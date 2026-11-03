

===== FILE: skills/detecting-bluetooth-low-energy-attacks/SKILL.md =====
---
name: detecting-bluetooth-low-energy-attacks
description: 'Detects and analyzes Bluetooth Low Energy (BLE) security attacks including
  sniffing, replay attacks, GATT enumeration abuse, and Man-in-the-Middle interception.
  Uses Ubertooth One and nRF52840 sniffers for packet capture, the bleak Python library
  for GATT service enumeration, and crackle for BLE encryption cracking. Use when
  assessing IoT device BLE security, monitoring for BLE-based attacks on wireless
  infrastructure, or performing authorized BLE penetration testing. Activates for
  requests involving BLE security assessment, Ubertooth sniffing, GATT enumeration,
  or BLE replay detection.

  '
domain: cybersecurity
subdomain: wireless-security
author: mukul975
tags:
- ble
- bluetooth
- ubertooth
- nrf-sniffer
- gatt
- wireless-security
- iot-security
- replay-attack
version: 1.0.0
license: Apache-2.0
nist_csf:
- PR.IR-01
- DE.CM-01
- ID.AM-03
mitre_attack:
- T1011.001
- T1557
- T1040
- T1200
---
# Detecting Bluetooth Low Energy Attacks

## Disclaimer

This skill is intended for authorized security testing, penetration testing engagements, CTF competitions, and educational purposes only. Sniffing, intercepting, or manipulating Bluetooth communications without authorization may violate federal wiretapping laws and local regulations. Always obtain explicit written permission before conducting any wireless security assessment.

## When to Use

Use this skill when:
- Performing authorized BLE security assessments of IoT devices, medical devices, or smart locks
- Monitoring a wireless environment for BLE-based replay attacks, spoofing, or unauthorized enumeration
- Analyzing BLE packet captures to detect Man-in-the-Middle attacks or pairing exploitation
- Enumerating GATT services and characteristics to identify insecure read/write permissions on BLE peripherals
- Assessing BLE encryption strength and testing for crackable pairing exchanges
- Building BLE intrusion detection capabilities for wireless security monitoring

**Do not use** for intercepting BLE communications without explicit authorization. Do not deploy BLE scanning tools in environments where wireless monitoring is prohibited.

## Prerequisites

- Ubertooth One hardware for passive BLE sniffing, or Nordic nRF52840 USB Dongle with nRF Sniffer firmware
- Python 3.10+ with pip
- bleak library: `pip install bleak` (cross-platform BLE GATT client)
- Wireshark with BLE dissector plugins for packet analysis
- crackle tool for BLE encryption analysis: built from source at github.com/mikeryan/crackle
- ubertooth-btle CLI tools: `apt install ubertooth` (Linux) or build from source
- Bluetooth 4.0+ adapter on the host system for bleak-based scanning
- Linux recommended for full Ubertooth/nRF sniffer support

## Workflow

### Step 1: BLE Environment Discovery and Device Scanning

Scan the environment to identify BLE devices and their advertising data:

```bash
# Scan for BLE devices using bleak (cross-platform)
python -c "
import asyncio
from bleak import BleakScanner

async def scan():
    devices = await BleakScanner.discover(timeout=10.0)
    for d in devices:
        print(f'{d.address} | RSSI: {d.rssi} | Name: {d.name or \"Unknown\"}')
        for uuid in d.metadata.get('uuids', []):
            print(f'  Service: {uuid}')

asyncio.run(scan())
"

# Passive BLE sniffing with Ubertooth One (promiscuous mode)
ubertooth-btle -p -r capture.pcapng

# Follow a specific BLE connection
ubertooth-btle -f -t AA:BB:CC:DD:EE:FF -r connection.pcapng

# Use nRF Sniffer with Wireshark (via extcap interface)
wireshark -i nRF_Sniffer -k
```

### Step 2: GATT Service and Characteristic Enumeration

Connect to target BLE peripherals and enumerate their GATT profile:

```bash
# Enumerate all services, characteristics, and descriptors
python -c "
import asyncio
from bleak import BleakClient

async def enum_gatt(address):
    async with BleakClient(address) as client:
        print(f'Connected: {client.is_connected}')
        for service in client.services:
            print(f'Service: {service.uuid} - {service.description}')
            for char in service.characteristics:
                props = ','.join(char.properties)
                print(f'  Char: {char.uuid} | Props: {props}')
                for desc in char.descriptors:
                    val = await client.read_gatt_descriptor(desc.handle)
                    print(f'    Desc: {desc.uuid} = {val}')

asyncio.run(enum_gatt('AA:BB:CC:DD:EE:FF'))
"
```

Security-relevant findings during GATT enumeration:
- Characteristics with `write-without-response` or `write` without authentication
- Readable characteristics exposing device configuration, credentials, or firmware versions
- Missing Client Characteristic Configuration Descriptor (CCCD) protection on notification characteristics

### Step 3: BLE Packet Capture and Analysis

Capture BLE traffic for offline analysis:

```bash
# Capture with Ubertooth in PcapNG format (recommended)
ubertooth-btle -f -r capture.pcapng

# Capture in PCAP/PPI format for crackle compatibility
ubertooth-btle -f -c capture_ppi.pcap

# Analyze capture in Wireshark
wireshark capture.pcapng
# Apply display filter: btle
# Filter connection requests: btle.advertising_header.pdu_type == 0x05
# Filter data packets: btle.data_header

# Extract pairing information with tshark
tshark -r capture.pcapng -Y "btle.control_opcode == 0x01" -T fields \
  -e btle.master_bd_addr -e btle.slave_bd_addr
```

### Step 4: BLE Encryption Analysis with Crackle

Analyze captured pairing exchanges to test encryption strength:

```bash
# Crack BLE Legacy Pairing (Just Works / passkey)
crackle -i capture_ppi.pcap -o decrypted.pcap

# Crack with known Temporary Key (TK)
crackle -i capture_ppi.pcap -o decrypted.pcap -l 000000

# Analyze decrypted traffic
wireshark decrypted.pcap
```

BLE Legacy Pairing with Just Works mode uses a TK of all zeros, making it trivially
crackable. Passkey entry uses a 6-digit PIN (000000-999999) that can be brute-forced
in under a second. Only BLE Secure Connections (LE Secure Connections with ECDH)
provides adequate protection against passive eavesdropping.

### Step 5: Replay Attack Detection and Testing

Monitor for and test BLE replay attack susceptibility:

```bash
# Capture characteristic write operations
# Record the raw bytes written to a target characteristic
# Then replay the exact same bytes to test if the device accepts stale commands

python -c "
import asyncio
from bleak import BleakClient

TARGET = 'AA:BB:CC:DD:EE:FF'
CHAR_UUID = '0000fff1-0000-1000-8000-00805f9b34fb'

async def replay_test():
    async with BleakClient(TARGET) as client:
        # Step 1: Read current state
        val = await client.read_gatt_char(CHAR_UUID)
        print(f'Current value: {val.hex()}')

        # Step 2: Write a command (captured from previous session)
        captured_command = bytes.fromhex('0102030405')
        await client.write_gatt_char(CHAR_UUID, captured_command)
        print('Replayed captured command')

        # Step 3: Verify if command was accepted
        new_val = await client.read_gatt_char(CHAR_UUID)
        print(f'New value: {new_val.hex()}')
        if new_val != val:
            print('VULNERABLE: Device accepted replayed command')

asyncio.run(replay_test())
"
```

Indicators of replay vulnerability:
- Device accepts previously captured write commands without freshness validation
- No sequence number, timestamp, or challenge-response mechanism in the protocol
- Device state changes in response to replayed commands

### Step 6: Man-in-the-Middle Detection

Detect BLE MITM attacks by monitoring for anomalous behavior:

```bash
# Monitor for BLE address spoofing (device impersonation)
# Compare advertising data fingerprints over time

# Monitor for unexpected connection parameter changes
tshark -r capture.pcapng -Y "btle.control_opcode == 0x00" -T fields \
  -e btle.control.interval.min -e btle.control.interval.max

# Detect GATTacker/BTLEjuice MITM patterns:
# - Cloned advertising data with different BD_ADDR
# - Rapid connect/disconnect cycles on the same channel
# - Duplicate service UUIDs from different addresses

# Monitor for suspicious pairing requests
tshark -r capture.pcapng -Y "btl2cap.cid == 0x0006" -T fields \
  -e btsmp.opcode -e btsmp.io_capability -e btsmp.auth_req
```

### Step 7: Continuous BLE Security Monitoring

Deploy ongoing BLE monitoring for threat detection:

```bash
# Run the agent in monitoring mode
python agent.py --mode monitor --duration 3600 --output ble_alerts.json

# Combine with Ubertooth for passive monitoring
ubertooth-btle -p -r - | python agent.py --mode analyze --pcap-stdin

# Alert on specific threat indicators
python agent.py --mode monitor --alert-on replay,spoofing,weak-pairing
```

## Key Concepts

| Term | Definition |
|------|-----------|
| **BLE (Bluetooth Low Energy)** | Low-power wireless protocol (Bluetooth 4.0+) optimized for IoT devices, operating on 2.4 GHz with 40 channels (3 advertising, 37 data) |
| **GATT (Generic Attribute Profile)** | BLE data model organizing device capabilities into services, characteristics, and descriptors; the primary interface for reading/writing BLE device data |
| **Ubertooth One** | Open-source 2.4 GHz wireless development platform capable of passive BLE and Bluetooth Classic sniffing across all BLE channels |
| **nRF Sniffer** | Nordic Semiconductor firmware for nRF52840 USB dongle that enables BLE packet capture with Wireshark integration via extcap |
| **Replay Attack** | Attack where previously captured BLE commands are retransmitted to a device to trigger unauthorized actions without knowledge of encryption keys |
| **Just Works Pairing** | BLE Legacy Pairing method using TK=0 with no user confirmation, providing zero protection against passive eavesdropping and MITM attacks |
| **LE Secure Connections** | BLE 4.2+ pairing mode using ECDH key exchange (P-256 curve) that provides protection against passive eavesdropping; recommended over Legacy Pairing |
| **Crackle** | Open-source tool that exploits weaknesses in BLE Legacy Pairing to recover the Long Term Key (LTK) and decrypt captured BLE traffic |
| **GATTacker** | BLE MITM framework that clones a peripheral's GATT profile and advertising data, then relays traffic between the real device and the victim central |

## Tools & Systems

- **Ubertooth One + ubertooth-btle**: Hardware sniffer and CLI tool for passive BLE packet capture in pcapng/pcap format
- **nRF52840 USB Dongle + nRF Sniffer**: Nordic Semiconductor BLE sniffer with native Wireshark extcap integration
- **bleak**: Cross-platform Python asyncio BLE GATT client library for device scanning, connection, and characteristic read/write
- **crackle**: BLE Legacy Pairing encryption cracker that recovers LTK from captured pairing exchanges
- **Wireshark**: Network protocol analyzer with BLE/BTLE dissectors for packet-level inspection of captured traffic
- **GATTacker / BTLEjuice**: BLE Man-in-the-Middle frameworks for intercepting and modifying BLE traffic between central and peripheral
- **tshark**: Command-line Wireshark for scripted BLE packet extraction and field analysis

## Common Pitfalls

- **Ubertooth channel hopping limitations**: Ubertooth follows one connection at a time. If multiple BLE connections are active, you must target a specific device address with `-t` to follow its data channels.
- **BLE 5.0 extended advertising**: Devices using BLE 5.0 extended advertising on secondary channels may not be captured by older Ubertooth firmware. Update to the latest firmware.
- **bleak platform differences**: BLE scanning behavior varies across OS backends. On Linux, scanning requires root or appropriate capabilities. On macOS, device addresses are randomized UUIDs.
- **crackle requires Legacy Pairing**: crackle only works against BLE Legacy Pairing (Bluetooth 4.0/4.1). LE Secure Connections (4.2+) use ECDH and cannot be cracked with this approach.
- **BLE address randomization**: Many modern BLE devices use random resolvable private addresses (RPA) that rotate periodically, making device tracking and connection following more difficult.
- **Capture format matters**: Use PCAP with PPI headers (`-c` flag) for crackle compatibility. PcapNG (`-r` flag) is recommended for Wireshark analysis but not supported by crackle.

## Output Format

```
## Finding: BLE Smart Lock Accepts Replayed Unlock Commands

**ID**: BLE-001
**Severity**: Critical (CVSS 9.3)
**Device**: SmartLock-Pro (AA:BB:CC:DD:EE:FF)
**Attack Type**: Replay Attack

**Description**:
The BLE smart lock accepts previously captured GATT write commands
on characteristic 0000fff1-0000-1000-8000-00805f9b34fb without
any freshness validation. An attacker who captures a single unlock
command can replay it indefinitely to unlock the device.

**Proof of Concept**:
1. Capture unlock command: ubertooth-btle -f -t AA:BB:CC:DD:EE:FF -r capture.pcap
2. Extract write payload from characteristic fff1: 01 42 A3 7F 00
3. Replay via bleak: await client.write_gatt_char(CHAR_UUID, bytes.fromhex('0142a37f00'))
4. Lock disengages without re-authentication

**Impact**:
Any attacker within BLE range (~100m with directional antenna) who
captures a single unlock event can replay it to gain physical access
to the protected area indefinitely.

**Remediation**:
Implement challenge-response authentication with per-session nonces.
Each command should include a server-generated challenge that expires
after use. Use LE Secure Connections for pairing to prevent passive
capture of the pairing exchange.
```

===== END FILE =====


===== FILE: skills/detecting-bluetooth-low-energy-attacks/LICENSE =====

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


===== FILE: skills/detecting-bluetooth-low-energy-attacks/references/api-reference.md =====
# API Reference: BLE Attack Detection Agent

## Overview

Scans, enumerates, and analyzes Bluetooth Low Energy devices for security vulnerabilities including weak pairing, replay attack susceptibility, insecure GATT permissions, advertising spoofing, and Man-in-the-Middle indicators. Combines Ubertooth/nRF hardware sniffing with bleak-based GATT enumeration and crackle-based encryption analysis. For authorized wireless security testing only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| bleak | >=0.21 | Cross-platform asyncio BLE GATT client for scanning and enumeration |
| tshark | (system) | Command-line Wireshark for BLE packet extraction and field analysis |
| ubertooth-btle | (system) | Ubertooth One CLI for passive BLE sniffing and packet capture |
| crackle | (system) | BLE Legacy Pairing encryption cracker for LTK recovery |

## CLI Usage

```bash
# Scan for BLE devices in range
python agent.py --mode scan --scan-duration 15 --output scan_report.json

# Enumerate GATT services on a target device
python agent.py --mode enumerate --target AA:BB:CC:DD:EE:FF --output gatt_report.json

# Test replay vulnerability on a specific characteristic
python agent.py --mode replay --target AA:BB:CC:DD:EE:FF \
  --char-uuid 0000fff1-0000-1000-8000-00805f9b34fb \
  --replay-payload 0102030405 --output replay_report.json

# Monitor for BLE advertising spoofing
python agent.py --mode monitor --scan-duration 60 \
  --known-devices known.json --output monitor_report.json

# Analyze a BLE packet capture
python agent.py --mode analyze --pcap capture.pcapng --output pcap_report.json

# Full assessment with Ubertooth capture
python agent.py --mode full --target AA:BB:CC:DD:EE:FF \
  --ubertooth-capture 120 --pcap-format ppi \
  --char-uuid 0000fff1-0000-1000-8000-00805f9b34fb \
  --replay-payload 0102030405 --output full_report.json
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--mode` | No | Operating mode: `scan`, `enumerate`, `replay`, `monitor`, `analyze`, `full` (default: `scan`) |
| `--target` | Conditional | Target BLE device address (required for enumerate/replay modes) |
| `--scan-duration` | No | BLE scan duration in seconds (default: 10) |
| `--char-uuid` | Conditional | GATT characteristic UUID for replay testing |
| `--replay-payload` | Conditional | Hex-encoded payload for replay test |
| `--pcap` | Conditional | Path to BLE pcap/pcapng file for analysis mode |
| `--ubertooth-capture` | No | Capture with Ubertooth for N seconds; 0 to disable (default: 0) |
| `--pcap-format` | No | Ubertooth capture format: `pcapng`, `ppi`, `le` (default: `pcapng`) |
| `--known-devices` | No | JSON file mapping known device addresses to names for spoofing detection |
| `--output` | No | Output report file path (default: `ble_security_report.json`) |

## Key Functions

### `scan_ble_devices(scan_duration)`
Discovers BLE devices using bleak BleakScanner. Returns device address, name, RSSI, service UUIDs, manufacturer data, service data, and TX power for each device found.

### `enumerate_gatt_services(target_address, timeout)`
Connects to a BLE peripheral and enumerates all GATT services, characteristics, and descriptors. Reads characteristic values when readable. Flags writable characteristics, write-without-response properties, and characteristics containing sensitive keyword patterns.

### `test_replay_vulnerability(target_address, char_uuid, test_payload_hex, read_after)`
Writes a captured/test payload to a characteristic, then replays the same payload to detect if the device accepts stale commands without freshness validation. Reads state before and after to confirm replay effect.

### `detect_advertising_spoofing(scan_duration, known_devices)`
Monitors BLE advertising in real-time to detect spoofing indicators: same device name from multiple addresses (cloned device), known device names from unknown addresses (impersonation), and abnormal RSSI fluctuations (relay attack).

### `analyze_pcap_for_ble_attacks(pcap_path)`
Analyzes BLE packet captures using tshark and crackle. Detects Just Works pairing, Legacy Pairing without Secure Connections, excessive connection attempts, and attempts LTK recovery with crackle.

### `run_ubertooth_capture(output_path, target_address, duration, pcap_format)`
Starts a passive BLE capture using Ubertooth One in either promiscuous or follow mode. Supports pcapng, PPI (crackle-compatible), and LE pseudoheader output formats.

### `generate_report(scan_results, gatt_profiles, replay_results, spoofing_findings, pcap_findings, output_path)`
Aggregates all findings into a JSON report with severity breakdown and full device/GATT data.

## Threat Detection Coverage

| Threat | Detection Method | Finding ID |
|--------|-----------------|------------|
| Insecure GATT Permissions | GATT enumeration, property analysis | BLE-GATT-001/002/003 |
| Replay Attack | Payload write + re-write + state comparison | BLE-REPLAY-001 |
| Device Spoofing | Multi-address name monitoring | BLE-SPOOF-001/002/003 |
| Just Works Pairing | PCAP SMP opcode analysis | BLE-PAIR-001 |
| Legacy Pairing (No SC) | PCAP auth_req flag analysis | BLE-PAIR-002 |
| Weak Encryption | crackle LTK recovery | BLE-CRACK-001 |
| Connection Flooding | PCAP connection event counting | BLE-PCAP-002 |

===== END FILE =====


===== FILE: skills/detecting-bluetooth-low-energy-attacks/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized penetration testing and lab environments only
"""BLE Attack Detection Agent - Scans, enumerates, and analyzes Bluetooth Low Energy
devices for security vulnerabilities including weak pairing, replay susceptibility,
insecure GATT permissions, and advertising spoofing."""

import argparse
import asyncio
import json
import logging
import struct
import subprocess
import sys
import time
from collections import defaultdict
from datetime import datetime
from pathlib import Path

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

# Standard BLE service UUIDs for identification
KNOWN_SERVICES = {
    "00001800-0000-1000-8000-00805f9b34fb": "Generic Access",
    "00001801-0000-1000-8000-00805f9b34fb": "Generic Attribute",
    "0000180a-0000-1000-8000-00805f9b34fb": "Device Information",
    "0000180f-0000-1000-8000-00805f9b34fb": "Battery Service",
    "00001809-0000-1000-8000-00805f9b34fb": "Health Thermometer",
    "0000180d-0000-1000-8000-00805f9b34fb": "Heart Rate",
    "00001812-0000-1000-8000-00805f9b34fb": "HID (Human Interface Device)",
    "0000fee0-0000-1000-8000-00805f9b34fb": "Firmware Update Service",
    "0000fff0-0000-1000-8000-00805f9b34fb": "Vendor-Specific Control",
}

# Properties that indicate potential security concerns
WRITABLE_PROPS = {"write", "write-without-response"}
READABLE_PROPS = {"read"}
NOTIFY_PROPS = {"notify", "indicate"}


async def scan_ble_devices(scan_duration=10.0):
    """Scan for BLE devices and collect advertising data."""
    try:
        from bleak import BleakScanner
    except ImportError:
        logger.error("bleak not installed: pip install bleak")
        return []

    logger.info("Scanning for BLE devices (%0.1fs)...", scan_duration)
    devices_found = []

    devices = await BleakScanner.discover(timeout=scan_duration, return_adv=True)

    for address, (device, adv_data) in devices.items():
        device_info = {
            "address": address,
            "name": device.name or "Unknown",
            "rssi": adv_data.rssi,
            "service_uuids": adv_data.service_uuids or [],
            "manufacturer_data": {
                str(k): v.hex() for k, v in (adv_data.manufacturer_data or {}).items()
            },
            "service_data": {
                k: v.hex() for k, v in (adv_data.service_data or {}).items()
            },
            "tx_power": adv_data.tx_power,
            "connectable": getattr(adv_data, "connectable", None),
        }
        devices_found.append(device_info)
        logger.info("Found: %s (%s) RSSI: %d dBm", device.name or "Unknown", address, adv_data.rssi)

    logger.info("Scan complete: %d devices found", len(devices_found))
    return devices_found


async def enumerate_gatt_services(target_address, timeout=30.0):
    """Connect to a BLE device and enumerate all GATT services, characteristics, and descriptors."""
    try:
        from bleak import BleakClient
    except ImportError:
        logger.error("bleak not installed: pip install bleak")
        return None

    gatt_profile = {
        "address": target_address,
        "services": [],
        "security_findings": [],
    }

    try:
        async with BleakClient(target_address, timeout=timeout) as client:
            if not client.is_connected:
                logger.error("Failed to connect to %s", target_address)
                return gatt_profile

            logger.info("Connected to %s", target_address)

            for service in client.services:
                svc_name = KNOWN_SERVICES.get(service.uuid, "Custom/Vendor Service")
                service_info = {
                    "uuid": service.uuid,
                    "name": svc_name,
                    "characteristics": [],
                }

                for char in service.characteristics:
                    char_info = {
                        "uuid": char.uuid,
                        "properties": list(char.properties),
                        "handle": char.handle,
                        "descriptors": [],
                        "value": None,
                    }

                    # Read characteristic value if readable
                    if READABLE_PROPS & set(char.properties):
                        try:
                            value = await client.read_gatt_char(char.uuid)
                            char_info["value"] = value.hex()

                            # Check for sensitive data exposure
                            try:
                                decoded = value.decode("utf-8", errors="ignore")
                                if any(kw in decoded.lower() for kw in
                                       ["password", "key", "token", "secret", "admin"]):
                                    gatt_profile["security_findings"].append({
                                        "id": "BLE-GATT-001",
                                        "severity": "High",
                                        "title": "Sensitive Data in Readable Characteristic",
                                        "detail": f"Characteristic {char.uuid} contains potentially "
                                                  f"sensitive data readable without authentication: "
                                                  f"{decoded[:50]}",
                                    })
                            except Exception:
                                pass
                        except Exception as e:
                            char_info["value"] = f"read_error: {e}"

                    # Flag writable characteristics without authentication
                    if WRITABLE_PROPS & set(char.properties):
                        gatt_profile["security_findings"].append({
                            "id": "BLE-GATT-002",
                            "severity": "Medium",
                            "title": "Writable Characteristic Without Authentication",
                            "detail": f"Characteristic {char.uuid} in service {svc_name} "
                                      f"allows write operations ({', '.join(char.properties)}). "
                                      "Verify authentication is enforced at the application layer.",
                        })

                    # Flag write-without-response (no confirmation)
                    if "write-without-response" in char.properties:
                        gatt_profile["security_findings"].append({
                            "id": "BLE-GATT-003",
                            "severity": "Medium",
                            "title": "Write-Without-Response Characteristic",
                            "detail": f"Characteristic {char.uuid} supports write-without-response. "
                                      "Commands sent to this characteristic have no delivery "
                                      "confirmation, making replay attacks harder to detect.",
                        })

                    # Read descriptors
                    for desc in char.descriptors:
                        try:
                            desc_val = await client.read_gatt_descriptor(desc.handle)
                            char_info["descriptors"].append({
                                "uuid": desc.uuid,
                                "handle": desc.handle,
                                "value": desc_val.hex(),
                            })
                        except Exception:
                            char_info["descriptors"].append({
                                "uuid": desc.uuid,
                                "handle": desc.handle,
                                "value": "read_error",
                            })

                    service_info["characteristics"].append(char_info)
                gatt_profile["services"].append(service_info)

            logger.info("Enumerated %d services, %d findings",
                        len(gatt_profile["services"]),
                        len(gatt_profile["security_findings"]))

    except Exception as e:
        logger.error("GATT enumeration failed for %s: %s", target_address, e)
        gatt_profile["error"] = str(e)

    return gatt_profile


async def test_replay_vulnerability(target_address, char_uuid, test_payload_hex, read_after=True):
    """Test if a BLE characteristic is vulnerable to replay attacks."""
    try:
        from bleak import BleakClient
    except ImportError:
        logger.error("bleak not installed: pip install bleak")
        return None

    result = {
        "target": target_address,
        "characteristic": char_uuid,
        "test_payload": test_payload_hex,
        "vulnerable": False,
        "detail": "",
    }

    payload = bytes.fromhex(test_payload_hex)

    try:
        async with BleakClient(target_address, timeout=30) as client:
            if not client.is_connected:
                result["detail"] = "Connection failed"
                return result

            # Read initial state if possible
            initial_value = None
            if read_after:
                try:
                    initial_value = await client.read_gatt_char(char_uuid)
                    logger.info("Initial value: %s", initial_value.hex())
                except Exception:
                    pass

            # Write the captured/test payload
            try:
                await client.write_gatt_char(char_uuid, payload)
                logger.info("Wrote replay payload: %s", test_payload_hex)
            except Exception as e:
                result["detail"] = f"Write rejected: {e}"
                return result

            # Small delay for device to process
            await asyncio.sleep(0.5)

            # Write the same payload again (replay)
            try:
                await client.write_gatt_char(char_uuid, payload)
                logger.info("Replayed same payload successfully")
                result["replay_accepted"] = True
            except Exception as e:
                result["detail"] = f"Replay rejected: {e}"
                result["replay_accepted"] = False
                return result

            # Read final state to check if replay had effect
            if read_after:
                try:
                    final_value = await client.read_gatt_char(char_uuid)
                    logger.info("Final value: %s", final_value.hex())
                    if initial_value and final_value != initial_value:
                        result["vulnerable"] = True
                        result["detail"] = (
                            "Device accepted replayed command and state changed. "
                            "No freshness validation detected."
                        )
                    else:
                        result["detail"] = "Replay accepted but no observable state change."
                        result["vulnerable"] = True  # Still accepted, just no visible effect
                except Exception:
                    result["vulnerable"] = True
                    result["detail"] = "Replay accepted; could not verify state change."
            else:
                result["vulnerable"] = True
                result["detail"] = "Replay payload accepted without error."

    except Exception as e:
        result["detail"] = f"Test failed: {e}"

    return result


async def detect_advertising_spoofing(scan_duration=30.0, known_devices=None):
    """Monitor BLE advertising for spoofing indicators."""
    try:
        from bleak import BleakScanner
    except ImportError:
        logger.error("bleak not installed: pip install bleak")
        return []

    findings = []
    device_history = defaultdict(list)

    logger.info("Monitoring BLE advertising for spoofing (%0.1fs)...", scan_duration)

    def detection_callback(device, advertisement_data):
        key = device.name or device.address
        entry = {
            "address": device.address,
            "rssi": advertisement_data.rssi,
            "timestamp": time.time(),
            "service_uuids": advertisement_data.service_uuids or [],
            "manufacturer_data": {
                str(k): v.hex() for k, v in (advertisement_data.manufacturer_data or {}).items()
            },
        }
        device_history[key].append(entry)

    scanner = BleakScanner(detection_callback=detection_callback)
    await scanner.start()
    await asyncio.sleep(scan_duration)
    await scanner.stop()

    # Analyze for spoofing indicators
    for name, entries in device_history.items():
        addresses = set(e["address"] for e in entries)

        # Multiple addresses with same name (possible spoofing)
        if len(addresses) > 1 and name != "Unknown":
            findings.append({
                "id": "BLE-SPOOF-001",
                "severity": "High",
                "title": "Multiple Addresses for Same Device Name",
                "detail": f"Device '{name}' advertised from {len(addresses)} different "
                          f"addresses: {', '.join(addresses)}. This may indicate address "
                          "spoofing or a cloned device (GATTacker-style MITM).",
                "addresses": list(addresses),
            })

        # Check for known device impersonation
        if known_devices:
            for entry in entries:
                if entry["address"] not in known_devices and name in known_devices.values():
                    findings.append({
                        "id": "BLE-SPOOF-002",
                        "severity": "Critical",
                        "title": "Known Device Name from Unknown Address",
                        "detail": f"Device '{name}' is advertising from unknown address "
                                  f"{entry['address']}. Expected address for this device "
                                  f"is in the known device list. Possible impersonation.",
                    })

        # Rapid RSSI fluctuations (possible relay attack)
        if len(entries) >= 5:
            rssi_values = [e["rssi"] for e in entries]
            rssi_range = max(rssi_values) - min(rssi_values)
            if rssi_range > 40:
                findings.append({
                    "id": "BLE-SPOOF-003",
                    "severity": "Medium",
                    "title": "Abnormal RSSI Fluctuation",
                    "detail": f"Device '{name}' ({entries[0]['address']}) shows RSSI range "
                              f"of {rssi_range} dBm (min: {min(rssi_values)}, max: "
                              f"{max(rssi_values)}). Large fluctuations may indicate a "
                              "relay attack or signal amplification.",
                })

    logger.info("Spoofing detection complete: %d findings", len(findings))
    return findings


def analyze_pcap_for_ble_attacks(pcap_path):
    """Analyze a BLE packet capture file for attack indicators using tshark."""
    findings = []

    if not Path(pcap_path).exists():
        logger.error("PCAP file not found: %s", pcap_path)
        return findings

    # Check for Legacy Pairing (vulnerable to crackle)
    try:
        result = subprocess.run(
            ["tshark", "-r", pcap_path, "-Y", "btsmp.opcode == 0x01",
             "-T", "fields", "-e", "btsmp.io_capability", "-e", "btsmp.auth_req"],
            capture_output=True, text=True, timeout=60,
        )
        if result.stdout.strip():
            lines = result.stdout.strip().split("\n")
            for line in lines:
                parts = line.split("\t")
                io_cap = parts[0] if len(parts) > 0 else ""
                auth_req = parts[1] if len(parts) > 1 else ""

                # io_capability 0x03 = NoInputNoOutput (Just Works)
                if io_cap == "0x03" or io_cap == "3":
                    findings.append({
                        "id": "BLE-PAIR-001",
                        "severity": "Critical",
                        "title": "BLE Just Works Pairing Detected",
                        "detail": "Pairing exchange uses NoInputNoOutput IO capability "
                                  "(Just Works). TK=0, trivially crackable with crackle. "
                                  "No MITM protection.",
                    })

                # Check if Secure Connections flag is not set
                if auth_req and not (int(auth_req, 0) & 0x08):
                    findings.append({
                        "id": "BLE-PAIR-002",
                        "severity": "High",
                        "title": "BLE Legacy Pairing (No Secure Connections)",
                        "detail": "Pairing uses Legacy Pairing without SC flag. "
                                  "Vulnerable to passive eavesdropping and LTK recovery "
                                  "via crackle tool.",
                    })
    except FileNotFoundError:
        logger.warning("tshark not found; skipping pcap pairing analysis")
    except subprocess.TimeoutExpired:
        logger.warning("tshark analysis timed out")

    # Count unique connection events
    try:
        result = subprocess.run(
            ["tshark", "-r", pcap_path, "-Y",
             "btle.advertising_header.pdu_type == 0x05",
             "-T", "fields", "-e", "btle.master_bd_addr", "-e", "btle.slave_bd_addr"],
            capture_output=True, text=True, timeout=60,
        )
        if result.stdout.strip():
            connections = result.stdout.strip().split("\n")
            unique_pairs = set()
            for conn in connections:
                unique_pairs.add(conn.strip())

            findings.append({
                "id": "BLE-PCAP-001",
                "severity": "Informational",
                "title": "BLE Connection Events Summary",
                "detail": f"Captured {len(connections)} connection requests across "
                          f"{len(unique_pairs)} unique device pairs.",
            })

            # Multiple rapid connections to same device (possible attack)
            if len(connections) > 10 and len(unique_pairs) < 3:
                findings.append({
                    "id": "BLE-PCAP-002",
                    "severity": "Medium",
                    "title": "Excessive Connection Attempts",
                    "detail": f"{len(connections)} connection attempts to "
                              f"{len(unique_pairs)} devices. May indicate brute-force "
                              "pairing or denial-of-service attack.",
                })
    except (FileNotFoundError, subprocess.TimeoutExpired):
        pass

    # Attempt crackle analysis
    try:
        result = subprocess.run(
            ["crackle", "-i", pcap_path],
            capture_output=True, text=True, timeout=120,
        )
        if "LTK" in result.stdout or "key" in result.stdout.lower():
            findings.append({
                "id": "BLE-CRACK-001",
                "severity": "Critical",
                "title": "BLE Encryption Key Recovered",
                "detail": f"crackle successfully recovered encryption key from captured "
                          f"pairing exchange. Encrypted traffic can be decrypted. "
                          f"Output: {result.stdout[:200]}",
            })
        elif "LE Secure Connections" in result.stdout:
            findings.append({
                "id": "BLE-CRACK-002",
                "severity": "Informational",
                "title": "LE Secure Connections Detected",
                "detail": "Pairing uses LE Secure Connections (ECDH). Not vulnerable "
                          "to crackle-based key recovery.",
            })
    except FileNotFoundError:
        logger.info("crackle not installed; skipping encryption analysis")
    except subprocess.TimeoutExpired:
        logger.warning("crackle analysis timed out")

    logger.info("PCAP analysis complete: %d findings", len(findings))
    return findings


def run_ubertooth_capture(output_path, target_address=None, duration=60, pcap_format="pcapng"):
    """Start a BLE packet capture with Ubertooth One."""
    cmd = ["ubertooth-btle"]

    if target_address:
        cmd.extend(["-f", "-t", target_address])  # Follow mode targeting specific device
    else:
        cmd.append("-p")  # Promiscuous mode

    if pcap_format == "pcapng":
        cmd.extend(["-r", output_path])
    elif pcap_format == "ppi":
        cmd.extend(["-c", output_path])  # PCAP/PPI for crackle compatibility
    else:
        cmd.extend(["-q", output_path])  # PCAP with LE pseudoheader

    logger.info("Starting Ubertooth capture: %s", " ".join(cmd))
    logger.info("Capturing for %d seconds...", duration)

    try:
        proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        time.sleep(duration)
        proc.terminate()
        proc.wait(timeout=10)
        logger.info("Capture saved to %s", output_path)
        return True
    except FileNotFoundError:
        logger.error("ubertooth-btle not found. Install: apt install ubertooth")
        return False
    except Exception as e:
        logger.error("Ubertooth capture failed: %s", e)
        return False


def generate_report(scan_results, gatt_profiles, replay_results, spoofing_findings,
                    pcap_findings, output_path):
    """Generate comprehensive BLE security assessment report."""
    all_findings = []

    # Collect GATT findings
    for profile in gatt_profiles:
        all_findings.extend(profile.get("security_findings", []))

    # Collect other findings
    for result in replay_results:
        if result and result.get("vulnerable"):
            all_findings.append({
                "id": "BLE-REPLAY-001",
                "severity": "Critical",
                "title": "Replay Attack Vulnerability",
                "detail": f"Device {result['target']} characteristic {result['characteristic']} "
                          f"is vulnerable to replay attacks. {result.get('detail', '')}",
            })

    all_findings.extend(spoofing_findings)
    all_findings.extend(pcap_findings)

    critical = [f for f in all_findings if f.get("severity") == "Critical"]
    high = [f for f in all_findings if f.get("severity") == "High"]
    medium = [f for f in all_findings if f.get("severity") == "Medium"]

    report = {
        "assessment": "BLE Security Assessment",
        "timestamp": datetime.utcnow().isoformat(),
        "devices_scanned": len(scan_results),
        "devices_enumerated": len(gatt_profiles),
        "summary": {
            "total_findings": len(all_findings),
            "critical": len(critical),
            "high": len(high),
            "medium": len(medium),
            "informational": len(all_findings) - len(critical) - len(high) - len(medium),
        },
        "scan_results": scan_results,
        "gatt_profiles": gatt_profiles,
        "replay_tests": replay_results,
        "findings": all_findings,
    }

    with open(output_path, "w") as f:
        json.dump(report, f, indent=2, default=str)
    logger.info("Report saved to %s (%d findings)", output_path, len(all_findings))
    return report


def main():
    parser = argparse.ArgumentParser(description="BLE Attack Detection Agent")
    parser.add_argument("--mode", choices=["scan", "enumerate", "replay", "monitor",
                                           "analyze", "full"],
                        default="scan", help="Operating mode")
    parser.add_argument("--target", help="Target BLE device address (AA:BB:CC:DD:EE:FF)")
    parser.add_argument("--scan-duration", type=float, default=10.0,
                        help="BLE scan duration in seconds (default: 10)")
    parser.add_argument("--char-uuid", help="Target GATT characteristic UUID for replay test")
    parser.add_argument("--replay-payload", help="Hex payload for replay test (e.g., 0102030405)")
    parser.add_argument("--pcap", help="Path to BLE pcap/pcapng file for analysis")
    parser.add_argument("--ubertooth-capture", type=int, default=0,
                        help="Capture with Ubertooth for N seconds (0=disabled)")
    parser.add_argument("--pcap-format", choices=["pcapng", "ppi", "le"],
                        default="pcapng", help="Ubertooth capture format")
    parser.add_argument("--known-devices", help="JSON file mapping known device addresses to names")
    parser.add_argument("--output", default="ble_security_report.json",
                        help="Output report file path")
    args = parser.parse_args()

    scan_results = []
    gatt_profiles = []
    replay_results = []
    spoofing_findings = []
    pcap_findings = []

    # Load known devices for spoofing detection
    known_devices = None
    if args.known_devices:
        try:
            with open(args.known_devices) as f:
                known_devices = json.load(f)
        except Exception as e:
            logger.warning("Could not load known devices: %s", e)

    # Ubertooth capture
    if args.ubertooth_capture > 0:
        capture_path = args.pcap or "ubertooth_capture.pcapng"
        run_ubertooth_capture(capture_path, args.target, args.ubertooth_capture, args.pcap_format)
        if not args.pcap:
            args.pcap = capture_path

    if args.mode in ("scan", "full"):
        scan_results = asyncio.run(scan_ble_devices(args.scan_duration))

    if args.mode in ("enumerate", "full") and args.target:
        profile = asyncio.run(enumerate_gatt_services(args.target))
        if profile:
            gatt_profiles.append(profile)

    if args.mode in ("replay", "full") and args.target and args.char_uuid and args.replay_payload:
        result = asyncio.run(
            test_replay_vulnerability(args.target, args.char_uuid, args.replay_payload)
        )
        if result:
            replay_results.append(result)

    if args.mode in ("monitor", "full"):
        spoofing_findings = asyncio.run(
            detect_advertising_spoofing(args.scan_duration, known_devices)
        )

    if args.mode in ("analyze", "full") and args.pcap:
        pcap_findings = analyze_pcap_for_ble_attacks(args.pcap)

    report = generate_report(scan_results, gatt_profiles, replay_results,
                             spoofing_findings, pcap_findings, args.output)

    print(json.dumps(report["summary"], indent=2))


if __name__ == "__main__":
    main()

===== END FILE =====


===== FILE: skills/performing-bluetooth-security-assessment/SKILL.md =====
---
name: performing-bluetooth-security-assessment
description: Assess Bluetooth Low Energy (BLE) device security using Python's bleak asyncio
  library to discover nearby devices, enumerate GATT services and characteristics, and flag
  unencrypted or unauthenticated read/write access to sensitive data. Use when auditing IoT,
  healthcare, fitness, or smart-home BLE devices for weak pairing configurations or known
  vulnerable device fingerprints.
domain: cybersecurity
subdomain: wireless-security
tags:
- bluetooth
- ble
- gatt
- wireless-security
version: '1.0'
author: mahipal
license: Apache-2.0
nist_csf:
- PR.IR-01
- DE.CM-01
- ID.AM-03
mitre_attack:
- T1557
- T1040
---


# Performing Bluetooth Security Assessment

## Overview

This skill covers performing Bluetooth Low Energy (BLE) security assessments using the Python bleak library. BLE devices are ubiquitous in IoT, healthcare, fitness, and smart home applications, and many ship with weak or absent security controls. This assessment identifies unencrypted GATT characteristics, devices broadcasting sensitive data, known vulnerable device fingerprints, and improperly secured pairing configurations.

The agent uses bleak's asyncio API to discover nearby BLE devices, connect to target devices, enumerate all GATT services and characteristics, and analyze security properties of each characteristic. It flags characteristics that allow unauthenticated read/write access to sensitive data and identifies devices matching known vulnerable profiles.


## When to Use

- When conducting security assessments that involve performing bluetooth security assessment
- When following incident response procedures for related security events
- When performing scheduled security testing or auditing activities
- When validating security controls through hands-on testing

## Prerequisites

- Python 3.9 or later
- bleak library (`pip install bleak`)
- Bluetooth adapter supporting BLE (Bluetooth 4.0+)
- Linux: BlueZ 5.43+ with D-Bus permissions
- Windows: Windows 10 version 1709+ with Bluetooth support
- macOS: macOS 10.15+ with CoreBluetooth

## Steps

1. **Scan for BLE devices**: Use BleakScanner to discover all advertising BLE devices within range. Capture device name, address (MAC), RSSI signal strength, and advertised service UUIDs.

2. **Identify target devices**: Filter discovered devices by name pattern, address, or minimum signal strength. Flag devices broadcasting default or well-known vulnerable names.

3. **Connect and enumerate GATT services**: Use BleakClient to connect to the target device and iterate over all GATT services. For each service, record the UUID, description, and contained characteristics.

4. **Analyze characteristic properties**: For each characteristic, examine its properties (read, write, write-without-response, notify, indicate). Flag characteristics that expose read or write access without requiring authentication or encryption.

5. **Check for known vulnerable UUIDs**: Compare discovered service and characteristic UUIDs against a database of known vulnerable or sensitive services (Heart Rate, Blood Pressure, Device Information, Battery Level) that should require encryption.

6. **Detect unencrypted data exposure**: Attempt to read characteristics that should be protected. Successful unauthenticated reads of sensitive data indicate missing security controls.

7. **Generate security report**: Compile all findings into a structured JSON report with severity classifications and remediation recommendations.

## Expected Output

```json
{
  "assessment_type": "ble_security_audit",
  "target_device": {
    "name": "SmartBand-XR",
    "address": "AA:BB:CC:DD:EE:FF",
    "rssi": -42
  },
  "services_found": 5,
  "characteristics_found": 18,
  "findings": [
    {
      "severity": "high",
      "finding": "Heart Rate Measurement readable without encryption",
      "uuid": "00002a37-0000-1000-8000-00805f9b34fb",
      "properties": ["read", "notify"],
      "remediation": "Enable encryption requirement on characteristic"
    }
  ],
  "risk_score": 7.5
}
```

===== END FILE =====


===== FILE: skills/performing-bluetooth-security-assessment/LICENSE =====

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


===== FILE: skills/performing-bluetooth-security-assessment/references/api-reference.md =====
# BLE Security Assessment API Reference

## Bleak Python Library (v0.21+)

### Device Discovery
```python
from bleak import BleakScanner

# Scan with advertisement data
devices = await BleakScanner.discover(timeout=10.0, return_adv=True)
# Returns: dict[str, tuple[BLEDevice, AdvertisementData]]

# Find specific device
device = await BleakScanner.find_device_by_name("DeviceName", timeout=10.0)
device = await BleakScanner.find_device_by_address("AA:BB:CC:DD:EE:FF", timeout=10.0)
```

### GATT Client Operations
```python
from bleak import BleakClient

async with BleakClient(address, timeout=15.0) as client:
    # Enumerate services
    for service in client.services:
        print(service.uuid, service.description)
        for char in service.characteristics:
            print(char.uuid, char.properties, char.descriptors)

    # Read characteristic
    value = await client.read_gatt_char("00002a19-0000-1000-8000-00805f9b34fb")

    # Write characteristic
    await client.write_gatt_char(char_uuid, bytearray([0x01, 0x02]))

    # Subscribe to notifications
    await client.start_notify(char_uuid, callback)
    await client.stop_notify(char_uuid)
```

## Common GATT Service UUIDs

| UUID (16-bit) | Service Name |
|---------------|-------------|
| `0x180D` | Heart Rate |
| `0x1810` | Blood Pressure |
| `0x1808` | Glucose |
| `0x180F` | Battery Service |
| `0x180A` | Device Information |
| `0x1812` | Human Interface Device |
| `0x1811` | Alert Notification |
| `0x1802` | Immediate Alert |
| `0x1803` | Link Loss |

## BLE Security Modes

| Mode | Level | Description |
|------|-------|-------------|
| LE Security Mode 1 | Level 1 | No security (no auth, no encryption) |
| LE Security Mode 1 | Level 2 | Unauthenticated pairing with encryption |
| LE Security Mode 1 | Level 3 | Authenticated pairing with encryption |
| LE Security Mode 1 | Level 4 | Authenticated LE Secure Connections |
| LE Security Mode 2 | Level 1 | Unauthenticated data signing |
| LE Security Mode 2 | Level 2 | Authenticated data signing |

## Linux BlueZ Commands

```bash
# Scan for BLE devices
sudo hcitool lescan

# Device info
sudo hcitool leinfo AA:BB:CC:DD:EE:FF

# Interactive GATT tool
gatttool -b AA:BB:CC:DD:EE:FF -I
> connect
> primary          # List services
> characteristics  # List characteristics
> char-read-hnd 0x000e

# btmgmt commands
sudo btmgmt info
sudo btmgmt find -l
sudo btmgmt pair -c 3 -t 0 AA:BB:CC:DD:EE:FF
```

===== END FILE =====


===== FILE: skills/performing-bluetooth-security-assessment/scripts/agent.py =====
#!/usr/bin/env python3
"""BLE security assessment using bleak for device scanning and GATT enumeration."""

import argparse
import asyncio
import json
import sys
import time

from bleak import BleakClient, BleakScanner


SENSITIVE_SERVICE_UUIDS = {
    "0000180d-0000-1000-8000-00805f9b34fb": "Heart Rate",
    "00001810-0000-1000-8000-00805f9b34fb": "Blood Pressure",
    "00001808-0000-1000-8000-00805f9b34fb": "Glucose",
    "00001809-0000-1000-8000-00805f9b34fb": "Health Thermometer",
    "0000180f-0000-1000-8000-00805f9b34fb": "Battery Service",
    "0000180a-0000-1000-8000-00805f9b34fb": "Device Information",
    "00001812-0000-1000-8000-00805f9b34fb": "Human Interface Device",
    "00001802-0000-1000-8000-00805f9b34fb": "Immediate Alert",
}

SENSITIVE_CHAR_UUIDS = {
    "00002a37-0000-1000-8000-00805f9b34fb": "Heart Rate Measurement",
    "00002a35-0000-1000-8000-00805f9b34fb": "Blood Pressure Measurement",
    "00002a18-0000-1000-8000-00805f9b34fb": "Glucose Measurement",
    "00002a1c-0000-1000-8000-00805f9b34fb": "Temperature Measurement",
    "00002a19-0000-1000-8000-00805f9b34fb": "Battery Level",
    "00002a29-0000-1000-8000-00805f9b34fb": "Manufacturer Name",
    "00002a26-0000-1000-8000-00805f9b34fb": "Firmware Revision",
    "00002a28-0000-1000-8000-00805f9b34fb": "Software Revision",
    "00002a25-0000-1000-8000-00805f9b34fb": "Serial Number",
}

VULNERABLE_DEVICE_PATTERNS = [
    "ITAG", "SmartLock", "BLE_Door", "FitBand", "iTag",
    "CC2541", "HM-10", "JDY-08", "AT-09", "MLT-BT05",
]


async def scan_devices(scan_time: float) -> list:
    devices = await BleakScanner.discover(timeout=scan_time, return_adv=True)
    results = []
    for addr, (device, adv_data) in devices.items():
        name = adv_data.local_name or device.name or "Unknown"
        vuln_match = None
        for pattern in VULNERABLE_DEVICE_PATTERNS:
            if pattern.lower() in name.lower():
                vuln_match = pattern
                break
        results.append({
            "address": str(addr),
            "name": name,
            "rssi": adv_data.rssi,
            "service_uuids": [str(u) for u in (adv_data.service_uuids or [])],
            "manufacturer_data": {str(k): v.hex() for k, v in (adv_data.manufacturer_data or {}).items()},
            "known_vulnerable_pattern": vuln_match,
        })
    results.sort(key=lambda d: d["rssi"], reverse=True)
    return results


async def enumerate_gatt(device_address: str) -> dict:
    findings = []
    services_info = []
    total_chars = 0
    async with BleakClient(device_address, timeout=15.0) as client:
        if not client.is_connected:
            return {"error": f"Failed to connect to {device_address}"}
        for service in client.services:
            svc_uuid = str(service.uuid)
            svc_name = SENSITIVE_SERVICE_UUIDS.get(svc_uuid, service.description or "Unknown")
            is_sensitive_svc = svc_uuid in SENSITIVE_SERVICE_UUIDS
            chars_info = []
            for char in service.characteristics:
                total_chars += 1
                char_uuid = str(char.uuid)
                props = char.properties
                char_name = SENSITIVE_CHAR_UUIDS.get(char_uuid, char.description or "Unknown")
                is_sensitive_char = char_uuid in SENSITIVE_CHAR_UUIDS
                char_entry = {
                    "uuid": char_uuid,
                    "name": char_name,
                    "properties": list(props),
                    "handle": char.handle,
                }
                if is_sensitive_char and ("read" in props):
                    findings.append({
                        "severity": "high",
                        "finding": f"{char_name} readable without encryption",
                        "uuid": char_uuid,
                        "service": svc_name,
                        "properties": list(props),
                        "remediation": "Enable encryption requirement on characteristic",
                    })
                if "write-without-response" in props and is_sensitive_svc:
                    findings.append({
                        "severity": "critical",
                        "finding": f"{char_name} writable without response in sensitive service",
                        "uuid": char_uuid,
                        "service": svc_name,
                        "properties": list(props),
                        "remediation": "Remove write-without-response or require authenticated pairing",
                    })
                if "write" in props and not is_sensitive_svc:
                    findings.append({
                        "severity": "medium",
                        "finding": f"{char_name} writable without known authentication",
                        "uuid": char_uuid,
                        "service": svc_name,
                        "properties": list(props),
                        "remediation": "Verify write access requires bonded connection",
                    })
                chars_info.append(char_entry)
            services_info.append({
                "uuid": svc_uuid,
                "name": svc_name,
                "sensitive": is_sensitive_svc,
                "characteristics": chars_info,
            })
    severity_weights = {"critical": 10, "high": 7, "medium": 4, "low": 1}
    risk_total = sum(severity_weights.get(f["severity"], 0) for f in findings)
    risk_score = min(10.0, round(risk_total / max(len(findings), 1), 1))
    return {
        "services_found": len(services_info),
        "characteristics_found": total_chars,
        "services": services_info,
        "findings": findings,
        "risk_score": risk_score,
    }


async def run_audit(device_address: str, scan_time: float) -> dict:
    scan_results = await scan_devices(scan_time)
    target = None
    for dev in scan_results:
        if dev["address"].upper() == device_address.upper():
            target = dev
            break
    if not target:
        return {"error": f"Device {device_address} not found in scan", "scanned_devices": len(scan_results)}
    gatt_result = await enumerate_gatt(device_address)
    return {
        "assessment_type": "ble_security_audit",
        "target_device": target,
        **gatt_result,
    }


def main():
    parser = argparse.ArgumentParser(description="BLE Security Assessment Tool")
    parser.add_argument("--action", choices=["scan", "enumerate", "audit"],
                        required=True, help="Action to perform")
    parser.add_argument("--scan-time", type=float, default=10.0,
                        help="BLE scan duration in seconds")
    parser.add_argument("--device-address", type=str, default=None,
                        help="Target BLE device address (MAC or UUID)")
    parser.add_argument("--output", type=str, default=None,
                        help="Output JSON file path")
    args = parser.parse_args()

    if args.action in ("enumerate", "audit") and not args.device_address:
        print(json.dumps({"error": "Device address required for enumerate/audit"}))
        sys.exit(1)

    start = time.time()
    if args.action == "scan":
        result = asyncio.run(scan_devices(args.scan_time))
        output = {"action": "scan", "devices_found": len(result), "devices": result}
    elif args.action == "enumerate":
        result = asyncio.run(enumerate_gatt(args.device_address))
        output = {"action": "enumerate", "target": args.device_address, **result}
    elif args.action == "audit":
        output = asyncio.run(run_audit(args.device_address, args.scan_time))

    output["elapsed_seconds"] = round(time.time() - start, 2)
    report = json.dumps(output, indent=2)
    if args.output:
        with open(args.output, "w") as f:
            f.write(report)
    print(report)


if __name__ == "__main__":
    main()

===== END FILE =====
