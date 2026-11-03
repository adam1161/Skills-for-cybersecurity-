

===== FILE: skills/monitoring-scada-modbus-traffic-anomalies/SKILL.md =====
---
name: monitoring-scada-modbus-traffic-anomalies
description: 'Monitors Modbus TCP traffic on SCADA and ICS networks to detect anomalous
  function code usage, unauthorized register writes, and suspicious communication
  patterns. The analyst uses deep packet inspection with pymodbus, Scapy, and Zeek
  to baseline normal PLC/RTU communication behavior, then applies statistical and
  rule-based anomaly detection to identify reconnaissance, parameter manipulation,
  and denial-of-service attacks targeting Modbus devices on port 502. Activates for
  requests involving Modbus traffic analysis, SCADA network monitoring, ICS anomaly
  detection, PLC security monitoring, or OT network threat detection.

  '
domain: cybersecurity
subdomain: ot-security
tags:
- Modbus-TCP
- SCADA
- ICS-security
- deep-packet-inspection
- anomaly-detection
- OT-monitoring
version: 1.0.0
author: mukul975
license: Apache-2.0
nist_csf:
- PR.IR-01
- DE.CM-01
- ID.AM-05
mitre_attack:
- T0816
- T0836
- T0830
- T1595
---
# Monitoring SCADA Modbus Traffic Anomalies

## When to Use

- Monitoring OT/ICS networks for unauthorized Modbus commands targeting PLCs, RTUs, or HMIs
- Detecting reconnaissance activity such as Modbus device enumeration (function code 43, Read Device Identification)
- Identifying unauthorized write operations (function codes 05, 06, 15, 16) to coils and holding registers that could alter physical process parameters
- Baselining normal Modbus communication patterns and alerting on deviations in function code distribution, register access ranges, or timing intervals
- Investigating suspected sabotage or insider threats manipulating SCADA process values through Modbus register writes

**Do not use** on networks without authorization from the asset owner, for active injection or fuzzing against production SCADA systems, or as a replacement for safety-instrumented systems (SIS) that provide physical process protection.

## Prerequisites

- Network tap or SPAN port on the OT network segment carrying Modbus TCP traffic (port 502)
- Python 3.9+ with pymodbus (>=3.6), scapy (>=2.5), and pandas for traffic analysis
- Zeek (formerly Bro) installed with the Modbus protocol analyzer enabled for passive traffic logging
- Wireshark or tshark for initial packet capture and validation of Modbus frame structure
- A baseline period of normal operations (minimum 48-72 hours) to establish communication profiles per device pair
- Network diagram identifying Modbus master-slave relationships, device IP addresses, and expected function code usage

## Workflow

### Step 1: Capture and Parse Modbus TCP Traffic

Establish passive monitoring on the OT network segment and begin capturing Modbus TCP frames:

- **Configure network tap**: Position the monitoring interface on the SPAN port mirroring the VLAN carrying Modbus TCP traffic between HMI/SCADA servers and PLCs. Verify bidirectional traffic capture with `tcpdump -i eth0 port 502 -c 100 -w modbus_capture.pcap`.
- **Parse Modbus TCP frame structure**: Each Modbus TCP frame contains a 7-byte MBAP (Modbus Application Protocol) header followed by the PDU. The MBAP header includes:
  - Transaction Identifier (2 bytes): Matches requests to responses
  - Protocol Identifier (2 bytes): Always 0x0000 for Modbus
  - Length (2 bytes): Number of following bytes including Unit ID
  - Unit Identifier (1 byte): Slave device address (0-247)
- **Extract function codes with Scapy**: Use Scapy's Modbus contrib module to dissect captured packets and extract function codes, register addresses, and values:
  ```python
  from scapy.all import rdpcap, TCP
  from scapy.contrib.modbus import ModbusADURequest, ModbusADUResponse

  packets = rdpcap("modbus_capture.pcap")
  for pkt in packets:
      if pkt.haslayer(ModbusADURequest):
          adu = pkt[ModbusADURequest]
          print(f"Src: {pkt['IP'].src} -> Dst: {pkt['IP'].dst} "
                f"Unit: {adu.unitId} FuncCode: {adu.funcCode}")
  ```
- **Enable Zeek Modbus logging**: Configure Zeek with `@load policy/protocols/modbus/known-masters-slaves` to generate `modbus.log` entries containing timestamp, source/destination IPs, function code, and exception responses. This provides continuous passive logging without custom scripting.
- **Validate frame integrity**: Check for malformed Modbus frames where the MBAP length field does not match the actual PDU length, Protocol Identifier is not 0x0000, or Unit Identifier falls outside the expected range for the monitored network.

### Step 2: Baseline Normal Communication Patterns

Build a behavioral profile of legitimate Modbus traffic to distinguish normal operations from anomalies:

- **Catalog function code distribution**: Record the frequency of each function code per source-destination pair over the baseline period. In typical SCADA environments, read operations (FC 01-04) vastly outnumber write operations (FC 05, 06, 15, 16), often at ratios exceeding 100:1. A sudden increase in write function codes is a strong indicator of process manipulation.
  ```
  Normal baseline example (72-hour period):
  HMI (10.1.1.10) -> PLC (10.1.1.50):
    FC 03 (Read Holding Registers):  432,180 packets  (97.2%)
    FC 04 (Read Input Registers):     10,540 packets  (2.4%)
    FC 06 (Write Single Register):     1,780 packets  (0.4%)
    FC 16 (Write Multiple Registers):      0 packets  (0.0%)
    FC 43 (Read Device ID):               0 packets  (0.0%)
  ```
- **Map register address ranges**: Document which holding register and coil address ranges each master polls. PLCs typically expose specific register blocks for monitoring (e.g., registers 0-99 for process values, 100-199 for setpoints). Access to registers outside the documented range indicates reconnaissance or misconfiguration.
- **Establish timing profiles**: Calculate the polling interval (mean, standard deviation) for each master-slave pair. SCADA polling is highly periodic, typically 100ms to 5s intervals. Deviations greater than 3 standard deviations from the mean suggest network issues or injected traffic from a rogue master.
- **Identify authorized masters**: Record all IP addresses that initiate Modbus requests (master role). In a properly segmented OT network, only the HMI server and engineering workstation should act as Modbus masters. Any new source IP sending Modbus requests is immediately suspicious.
- **Register value ranges**: For critical process registers (temperatures, pressures, flow rates, setpoints), record the observed minimum, maximum, mean, and standard deviation during normal operations. Values outside the physical process bounds indicate either sensor failure or malicious manipulation.

### Step 3: Detect Function Code Anomalies

Apply rule-based and statistical detection to identify suspicious function code usage:

- **Unauthorized write detection**: Alert when a Modbus write function code (05, 06, 15, 16) originates from a source IP not in the authorized writers list, or when write operations exceed the baseline frequency threshold:
  ```python
  WRITE_FUNCTION_CODES = {5, 6, 15, 16}
  AUTHORIZED_WRITERS = {"10.1.1.10", "10.1.1.11"}  # HMI and engineering WS

  def check_unauthorized_write(src_ip, function_code):
      if function_code in WRITE_FUNCTION_CODES and src_ip not in AUTHORIZED_WRITERS:
          return {
              "alert": "UNAUTHORIZED_MODBUS_WRITE",
              "severity": "CRITICAL",
              "src_ip": src_ip,
              "function_code": function_code,
              "description": f"Write FC {function_code} from unauthorized source {src_ip}"
          }
      return None
  ```
- **Reconnaissance detection**: Function code 43 (Read Device Identification) and function code 08 (Diagnostics) are rarely used during normal operations. Any occurrence from a non-engineering workstation indicates device enumeration. Also detect sequential scanning where a single source queries multiple Unit IDs within a short window.
- **Exception response monitoring**: Modbus exception codes (01: Illegal Function, 02: Illegal Data Address, 03: Illegal Data Value) in responses indicate the master sent an invalid request. A burst of exception responses suggests fuzzing or protocol-level attacks:
  ```
  Exception response correlation:
  - Isolated exception (1-2 per hour): Normal operational error
  - Burst (>10 per minute): Active scanning or fuzzing attempt
  - Continuous (>100 per hour): Denial-of-service or tool malfunction
  ```
- **Forbidden function code detection**: Some environments prohibit certain function codes entirely. Function codes 07 (Read Exception Status), 08 (Diagnostics), 17 (Report Slave ID), and 43 (Read Device Identification) are diagnostic functions that should not appear in production SCADA traffic. Alert on any occurrence.
- **Function code frequency anomaly**: Calculate the chi-squared statistic comparing the observed function code distribution against the baseline distribution. A significant deviation (p < 0.01) triggers an alert even if no individual function code crosses its threshold.

### Step 4: Monitor Register Values for Process Manipulation

Detect attempts to manipulate physical process parameters through register value analysis:

- **Setpoint change monitoring**: Track all write operations to holding registers that control process setpoints (temperatures, pressures, valve positions, motor speeds). Alert when:
  - The new value exceeds the defined safe operating range
  - The rate of change exceeds physical process capabilities (e.g., temperature setpoint jumping 50 degrees in one write)
  - Multiple setpoints change simultaneously, which does not match normal operator behavior
  ```python
  REGISTER_LIMITS = {
      40001: {"name": "Reactor Temperature Setpoint", "min": 50, "max": 200, "unit": "C",
              "max_rate": 5},   # Max 5 degrees per write cycle
      40010: {"name": "Pump Speed", "min": 0, "max": 3600, "unit": "RPM",
              "max_rate": 200},  # Max 200 RPM change per cycle
      40020: {"name": "Valve Position", "min": 0, "max": 100, "unit": "%",
              "max_rate": 10},   # Max 10% per cycle
  }

  def check_register_value(register_addr, new_value, previous_value):
      if register_addr not in REGISTER_LIMITS:
          return None
      limits = REGISTER_LIMITS[register_addr]
      alerts = []
      if new_value < limits["min"] or new_value > limits["max"]:
          alerts.append({
              "alert": "REGISTER_VALUE_OUT_OF_RANGE",
              "severity": "CRITICAL",
              "register": register_addr,
              "name": limits["name"],
              "value": new_value,
              "range": f"{limits['min']}-{limits['max']} {limits['unit']}"
          })
      if previous_value is not None:
          rate = abs(new_value - previous_value)
          if rate > limits["max_rate"]:
              alerts.append({
                  "alert": "REGISTER_VALUE_EXCESSIVE_RATE",
                  "severity": "HIGH",
                  "register": register_addr,
                  "name": limits["name"],
                  "change": rate,
                  "max_allowed": limits["max_rate"]
              })
      return alerts if alerts else None
  ```
- **Coil state monitoring**: Track coil writes (FC 05, FC 15) that control discrete outputs (pumps on/off, valves open/close, breakers trip/close). Detect rapid toggling (more than N state changes per minute) which could indicate equipment damage attempts.
- **Register read pattern anomaly**: If a master begins reading register ranges it has never accessed before, this may indicate an attacker using a compromised HMI to map the PLC memory layout before launching a targeted write attack.
- **Correlation with process data**: Where available, compare Modbus register values against independent process sensors (e.g., historian data). Discrepancies between the Modbus-reported value and the independent sensor indicate either sensor spoofing or register manipulation.

### Step 5: Detect Network-Level Anomalies

Identify anomalies in communication patterns that may indicate man-in-the-middle, replay, or denial-of-service attacks:

- **Rogue master detection**: Alert when a new source IP initiates Modbus TCP connections to port 502 on any slave device. Maintain a whitelist of authorized master IPs and generate a critical alert for any connection from an unknown source:
  ```python
  AUTHORIZED_MASTERS = {"10.1.1.10", "10.1.1.11"}

  def detect_rogue_master(src_ip, dst_ip, dst_port):
      if dst_port == 502 and src_ip not in AUTHORIZED_MASTERS:
          return {
              "alert": "ROGUE_MODBUS_MASTER",
              "severity": "CRITICAL",
              "src_ip": src_ip,
              "target_slave": dst_ip,
              "description": "Unauthorized device initiating Modbus connection"
          }
      return None
  ```
- **Transaction ID anomaly**: Modbus TCP uses transaction IDs to match requests with responses. Under normal operation, transaction IDs increment sequentially per master. Detect:
  - Duplicate transaction IDs from different sources (replay attack indicator)
  - Transaction ID gaps or resets (session hijacking indicator)
  - Responses with transaction IDs that do not match any recent request (injected response)
- **Timing anomaly detection**: Calculate inter-packet arrival times for each master-slave pair. Flag deviations greater than 3 standard deviations using a sliding window:
  ```python
  import numpy as np
  from collections import defaultdict

  class TimingAnomalyDetector:
      def __init__(self, window_size=1000, threshold_sigma=3.0):
          self.windows = defaultdict(list)
          self.window_size = window_size
          self.threshold_sigma = threshold_sigma

      def check(self, src_ip, dst_ip, timestamp):
          key = (src_ip, dst_ip)
          window = self.windows[key]
          if len(window) > 0:
              interval = timestamp - window[-1]
              if len(window) >= 100:
                  mean = np.mean(np.diff(window[-100:]))
                  std = np.std(np.diff(window[-100:]))
                  if std > 0 and abs(interval - mean) > self.threshold_sigma * std:
                      return {
                          "alert": "TIMING_ANOMALY",
                          "severity": "MEDIUM",
                          "pair": f"{src_ip}->{dst_ip}",
                          "interval": interval,
                          "expected_mean": mean,
                          "deviation_sigma": abs(interval - mean) / std
                      }
          window.append(timestamp)
          if len(window) > self.window_size:
              window.pop(0)
          return None
  ```
- **Connection flood detection**: Monitor the rate of new TCP connections to port 502 per slave device. Modbus slaves typically handle 1-5 persistent connections. More than 10 connection attempts per minute to a single slave indicates a connection flood DoS or scanning activity.
- **Payload size anomaly**: Modbus PDU max size is 253 bytes. Alert on oversized frames that exceed protocol limits, as these may indicate buffer overflow exploitation attempts against vulnerable PLC firmware.

## Key Concepts

| Term | Definition |
|------|------------|
| **Modbus TCP** | An application-layer protocol encapsulating Modbus frames in TCP/IP, communicating on port 502. It uses a 7-byte MBAP header (transaction ID, protocol ID, length, unit ID) followed by the Modbus PDU containing the function code and data. |
| **Function Code** | A single-byte identifier in the Modbus PDU specifying the operation: read coils (01), read discrete inputs (02), read holding registers (03), read input registers (04), write single coil (05), write single register (06), write multiple coils (15), write multiple registers (16), diagnostics (08), and device identification (43). |
| **MBAP Header** | Modbus Application Protocol header used in Modbus TCP. Contains Transaction ID for request-response matching, Protocol ID (always 0x0000 for Modbus), Length of remaining bytes, and Unit Identifier for addressing slaves behind gateways. |
| **Holding Register** | A 16-bit read/write register in a Modbus slave addressed at range 40001-49999 (protocol address 0-9998). Used for setpoints, configuration, and control values that can be written by the master. Primary target for process manipulation attacks. |
| **Coil** | A single-bit read/write data element in a Modbus slave addressed at range 00001-09999. Controls discrete outputs (valves, pumps, breakers). Write operations (FC 05/15) to coils can directly affect physical equipment state. |
| **Deep Packet Inspection** | Analysis beyond TCP/IP headers into the Modbus application-layer payload to extract function codes, register addresses, and values. Required because standard firewalls only inspect IP/port, missing protocol-level attacks that use legitimate Modbus framing. |
| **Rogue Master** | An unauthorized device sending Modbus requests to slave devices. In OT environments, only designated HMI servers and engineering workstations should act as Modbus masters. A rogue master can read process data or write dangerous values to PLCs. |
| **Register Value Baseline** | The statistical profile (min, max, mean, standard deviation) of values observed in specific registers during normal operations. Deviations beyond physical process bounds indicate sensor failure or malicious manipulation. |

## Tools & Systems

- **pymodbus**: Python library for Modbus protocol implementation supporting TCP, RTU, and ASCII modes. Used for building custom Modbus clients/servers, packet parsing, and simulating master-slave communication in test environments.
- **Scapy (contrib.modbus)**: Packet manipulation framework with Modbus TCP dissector for crafting, parsing, and sniffing Modbus frames. Enables field-level access to MBAP headers, function codes, and register data in captured packets.
- **Zeek (formerly Bro)**: Network security monitor with native Modbus protocol analyzer that generates structured logs (modbus.log) for every Modbus transaction including function codes, register addresses, and exception responses.
- **Wireshark/tshark**: Network protocol analyzer with built-in Modbus TCP dissector for visual inspection of packet captures, filtering by function code (`modbus.func_code == 6`), and exporting specific fields for analysis.
- **GRFICSv2**: An open-source virtual ICS environment for security research featuring a simulated chemical process with Modbus-connected PLCs, HMI, and historian. Used for testing detection rules against realistic SCADA traffic.
- **Suricata**: Network IDS/IPS with Modbus protocol support via application-layer rules that can match on function codes, register addresses, and values for real-time alerting.

## Common Scenarios

### Scenario: Detecting Unauthorized Parameter Manipulation in a Water Treatment Plant

**Context**: A water treatment facility uses Modbus TCP to communicate between the SCADA server (10.1.1.10) and six PLCs controlling chemical dosing pumps, filtration valves, and flow meters. The security team deploys passive Modbus traffic monitoring after an industry advisory about attacks targeting water utilities.

**Approach**:
1. Deploy a network tap on the OT VLAN switch mirroring all port 502 traffic to the monitoring interface. Run Zeek with Modbus logging and the custom Python analyzer in parallel.
2. Establish a 72-hour baseline during normal operations, cataloging function code distribution, register access patterns, and polling intervals for all six master-slave pairs.
3. Baseline reveals the SCADA server only uses FC 03 (Read Holding Registers) and FC 06 (Write Single Register) to PLC-3 (chemical dosing), with writes occurring 2-4 times per day matching operator shift changes.
4. On day 5, the analyzer detects FC 16 (Write Multiple Registers) from 10.1.1.10 to PLC-3, a function code never seen in the baseline. The write targets registers 40050-40055, which control chlorine dosing rates.
5. Seconds later, a second alert fires: the chlorine dosing setpoint in register 40050 changed from 2.5 mg/L to 25.0 mg/L, exceeding the safe maximum of 4.0 mg/L defined in the register value limits.
6. Cross-referencing with IT network logs reveals the SCADA server was accessed via Remote Desktop from an unauthorized VPN connection 20 minutes before the anomalous Modbus traffic.
7. The operations team is notified, the chemical dosing PLC is placed in manual override, and the incident response team isolates the compromised SCADA server.

**Pitfalls**:
- Relying solely on IT-side network monitoring (firewall logs, IDS) that does not inspect Modbus application-layer content and would see only a normal TCP connection on port 502
- Not defining per-register safe operating ranges, which would miss the dangerous dosing rate change despite detecting the unusual function code
- Setting the baseline period too short (e.g., 4 hours) and missing legitimate but infrequent write operations that occur only during shift changes or maintenance windows
- Failing to correlate OT network anomalies with IT network events, missing the RDP session that was the actual attack vector

### Scenario: Identifying Modbus Device Enumeration from a Compromised Engineering Workstation

**Context**: A manufacturing plant's SOC observes unusual network activity from an engineering workstation (10.1.2.20) that is authorized for PLC programming. The OT security team uses Modbus traffic monitoring to determine if the workstation is being used for reconnaissance.

**Approach**:
1. Filter Modbus traffic logs for all activity from 10.1.2.20 over the past 24 hours and compare against the baseline communication profile for that workstation.
2. Baseline shows 10.1.2.20 communicates with PLC-1 (10.1.1.50) only during scheduled maintenance windows using FC 03 and FC 06, approximately 200 packets per session.
3. Anomaly detection identifies 10.1.2.20 sent FC 43 (Read Device Identification) to 15 different IP addresses on the OT VLAN within a 10-minute window, none of which it has previously communicated with.
4. Further analysis shows FC 03 read requests to register ranges 0-9999 in blocks of 125 registers per request, systematically mapping the entire register space of each PLC contacted.
5. The engineering workstation is isolated, forensic imaging initiated, and all Modbus communication from that IP is blocked at the OT firewall. The device identification responses captured reveal the PLC firmware versions that the attacker obtained.

**Pitfalls**:
- Not flagging the engineering workstation because it is in the authorized masters list, missing that its communication pattern deviated drastically from its baseline profile
- Not detecting sequential register scanning because each individual read request is a valid FC 03 operation; only the aggregate pattern reveals the reconnaissance
- Blocking the workstation before capturing forensic evidence of the attack scope and exfiltrated data

## Output Format

```
## Modbus Traffic Anomaly Report

**Monitoring Period**: 2026-03-15 00:00:00 UTC to 2026-03-15 23:59:59 UTC
**Network Segment**: OT VLAN 10 (10.1.1.0/24)
**Packets Analyzed**: 2,847,320
**Anomalies Detected**: 4

---

### Alert 1: Unauthorized Write Operation

**Timestamp**: 2026-03-15 14:23:17 UTC
**Severity**: CRITICAL
**Source**: 10.1.2.20 (Engineering Workstation)
**Destination**: 10.1.1.52 (PLC-3 Chemical Dosing)
**Function Code**: 16 (Write Multiple Registers)
**Registers**: 40050-40055
**Values Written**: [250, 100, 0, 1, 3600, 1]
**Baseline**: FC 16 never observed for this source-destination pair

**Context**: Register 40050 (Chlorine Dosing Rate) changed from 25 to 250
(safe range: 10-40). Register 40054 (Dosing Timer) changed from 1800 to 3600.
Combined effect would double chlorine concentration over extended period.

**Recommended Action**: Immediately verify physical process state. Isolate
source device. Check register values against expected setpoints with
plant operator.

---

### Alert 2: Device Enumeration Detected

**Timestamp**: 2026-03-15 14:20:05 to 14:20:47 UTC
**Severity**: HIGH
**Source**: 10.1.2.20
**Targets**: 10.1.1.50, 10.1.1.51, 10.1.1.52, 10.1.1.53, 10.1.1.54 (+10 more)
**Function Code**: 43 (Read Device Identification)
**Baseline**: FC 43 never observed from this source

**Context**: Sequential scanning of 15 devices in 42 seconds. Device
identification responses reveal PLC vendor, model, and firmware versions
for all scanned devices.

**Recommended Action**: Investigate source workstation for compromise
indicators. Block FC 43 from non-engineering subnets at OT firewall.
```

===== END FILE =====


===== FILE: skills/monitoring-scada-modbus-traffic-anomalies/LICENSE =====

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


===== FILE: skills/monitoring-scada-modbus-traffic-anomalies/references/api-reference.md =====
# API Reference: Modbus TCP Traffic Anomaly Detector

## Overview

Monitors SCADA/ICS Modbus TCP traffic for unauthorized function codes, register value manipulation, device enumeration, rogue masters, and timing anomalies. Supports pcap analysis and live network capture using Scapy with configurable baselines and register safety limits. For authorized OT/ICS security monitoring only.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| scapy | >=2.5 | Packet capture, parsing, and Modbus TCP dissection |
| numpy | >=1.24 | Statistical analysis for timing and value anomaly detection |

## CLI Usage

```bash
# Analyze a pcap file
python agent.py --pcap modbus_capture.pcap \
  --authorized-masters 10.1.1.10 10.1.1.11 \
  --authorized-writers 10.1.1.10 \
  --register-limits-file register_limits.json \
  --baseline-file baseline.json \
  --output report.json

# Build a baseline from pcap
python agent.py --pcap normal_traffic.pcap \
  --baseline-mode --baseline-file baseline.json

# Live capture on network interface
python agent.py --interface eth0 --duration 3600 \
  --authorized-masters 10.1.1.10 10.1.1.11 \
  --authorized-writers 10.1.1.10 \
  --baseline-file baseline.json \
  --output report.json
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--pcap` | No* | Path to pcap file containing Modbus TCP traffic |
| `--interface` | No* | Network interface for live packet capture |
| `--duration` | No | Live capture duration in seconds (default: 0 = indefinite) |
| `--authorized-masters` | No | Space-separated list of authorized Modbus master IPs |
| `--authorized-writers` | No | Space-separated list of IPs allowed to send write commands |
| `--register-limits-file` | No | JSON file defining safe value ranges per register address |
| `--baseline-file` | No | Path to load existing or save new baseline profile |
| `--baseline-mode` | No | Build baseline without generating alerts |
| `--output` | No | Output report path (default: `modbus_anomaly_report.json`) |

\* Either `--pcap` or `--interface` is required.

## Register Limits File Format

```json
{
  "40001": {
    "name": "Reactor Temperature Setpoint",
    "min": 50,
    "max": 200,
    "unit": "C",
    "max_rate": 5
  },
  "40010": {
    "name": "Pump Speed",
    "min": 0,
    "max": 3600,
    "unit": "RPM",
    "max_rate": 200
  }
}
```

## Key Classes

### `ModbusAnomalyDetector`
Main detection engine that processes packets and generates alerts.

**Methods:**
- `analyze_packet(src_ip, dst_ip, dst_port, raw_payload, timestamp)` - Analyze a single Modbus TCP packet for anomalies. Returns list of alert dictionaries.
- `generate_report()` - Generate JSON anomaly report sorted by severity.

### `ModbusBaseline`
Maintains statistical profiles of normal Modbus communication patterns.

**Methods:**
- `record(src_ip, dst_ip, function_code, timestamp, registers, values)` - Record a packet observation into the baseline.
- `get_fc_distribution(src_ip, dst_ip)` - Get function code frequency distribution for a master-slave pair.
- `get_timing_stats(src_ip, dst_ip)` - Get mean and standard deviation of inter-packet intervals.
- `get_register_stats(register_addr)` - Get min/max/mean/std for observed register values.
- `save(filepath)` / `load(filepath)` - Persist or restore baseline to/from JSON.

## Key Functions

### `parse_mbap_header(data)`
Parses the 7-byte Modbus Application Protocol header (transaction ID, protocol ID, length, unit ID). Returns None for invalid headers (protocol ID != 0 or insufficient data).

### `parse_modbus_pdu(data)`
Extracts function code, register addresses, values, and exception status from the Modbus PDU. Supports FC 01-06, 15, 16 request parsing.

### `analyze_pcap(pcap_file, detector)`
Loads a pcap file with Scapy, filters for port 502 TCP traffic, and passes each Modbus packet to the detector.

### `live_capture(interface, detector, duration)`
Starts real-time Scapy sniffing on the specified interface with a BPF filter for TCP port 502.

## Alert Types

| Alert | Severity | Trigger |
|-------|----------|---------|
| `ROGUE_MODBUS_MASTER` | CRITICAL | Connection from IP not in authorized masters list |
| `UNAUTHORIZED_MODBUS_WRITE` | CRITICAL | Write function code from IP not in authorized writers list |
| `REGISTER_VALUE_OUT_OF_RANGE` | CRITICAL | Register value outside defined safe operating range |
| `DEVICE_ENUMERATION` | HIGH/CRITICAL | Diagnostic function codes (FC 7, 8, 17, 43) detected |
| `NEW_FUNCTION_CODE` | HIGH | Function code never seen in baseline for this master-slave pair |
| `REGISTER_VALUE_EXCESSIVE_RATE` | HIGH | Register value change exceeds maximum allowed rate |
| `EXCEPTION_BURST` | HIGH | 10+ Modbus exceptions per minute from a single slave |
| `TIMING_ANOMALY` | MEDIUM | Inter-packet interval deviates >3 sigma from baseline mean |
| `MALFORMED_MBAP_HEADER` | MEDIUM | Frame with invalid MBAP header structure |

===== END FILE =====


===== FILE: skills/monitoring-scada-modbus-traffic-anomalies/scripts/agent.py =====
#!/usr/bin/env python3
# For authorized OT/ICS security monitoring only
"""Modbus TCP Traffic Anomaly Detector - Monitors SCADA networks for suspicious Modbus activity."""

import json
import logging
import argparse
import struct
import time
from datetime import datetime, timezone
from collections import defaultdict
from pathlib import Path

import numpy as np

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

MODBUS_PORT = 502
MBAP_HEADER_SIZE = 7

FUNCTION_CODE_NAMES = {
    1: "Read Coils",
    2: "Read Discrete Inputs",
    3: "Read Holding Registers",
    4: "Read Input Registers",
    5: "Write Single Coil",
    6: "Write Single Register",
    7: "Read Exception Status",
    8: "Diagnostics",
    15: "Write Multiple Coils",
    16: "Write Multiple Registers",
    17: "Report Slave ID",
    22: "Mask Write Register",
    23: "Read/Write Multiple Registers",
    43: "Read Device Identification",
}

WRITE_FUNCTION_CODES = {5, 6, 15, 16, 22, 23}
READ_FUNCTION_CODES = {1, 2, 3, 4}
DIAGNOSTIC_FUNCTION_CODES = {7, 8, 17, 43}


def parse_mbap_header(data):
    """Parse 7-byte Modbus Application Protocol header from raw TCP payload."""
    if len(data) < MBAP_HEADER_SIZE:
        return None
    transaction_id, protocol_id, length, unit_id = struct.unpack(">HHHB", data[:7])
    if protocol_id != 0:
        return None
    return {
        "transaction_id": transaction_id,
        "protocol_id": protocol_id,
        "length": length,
        "unit_id": unit_id,
    }


def parse_modbus_pdu(data):
    """Parse Modbus PDU to extract function code, register addresses, and values."""
    if len(data) < MBAP_HEADER_SIZE + 1:
        return None
    pdu = data[MBAP_HEADER_SIZE:]
    function_code = pdu[0]
    is_exception = function_code > 0x80
    result = {
        "function_code": function_code & 0x7F if is_exception else function_code,
        "is_exception": is_exception,
        "exception_code": pdu[1] if is_exception and len(pdu) > 1 else None,
        "raw_pdu": pdu.hex(),
    }
    fc = result["function_code"]
    if not is_exception and len(pdu) >= 5:
        if fc in (1, 2, 3, 4):
            result["start_address"] = struct.unpack(">H", pdu[1:3])[0]
            result["quantity"] = struct.unpack(">H", pdu[3:5])[0]
        elif fc == 5:
            result["coil_address"] = struct.unpack(">H", pdu[1:3])[0]
            result["coil_value"] = struct.unpack(">H", pdu[3:5])[0]
        elif fc == 6:
            result["register_address"] = struct.unpack(">H", pdu[1:3])[0]
            result["register_value"] = struct.unpack(">H", pdu[3:5])[0]
        elif fc == 16 and len(pdu) >= 7:
            result["start_address"] = struct.unpack(">H", pdu[1:3])[0]
            result["quantity"] = struct.unpack(">H", pdu[3:5])[0]
            byte_count = pdu[5]
            values = []
            for i in range(result["quantity"]):
                offset = 6 + i * 2
                if offset + 2 <= len(pdu):
                    values.append(struct.unpack(">H", pdu[offset:offset + 2])[0])
            result["values"] = values
        elif fc == 15 and len(pdu) >= 6:
            result["start_address"] = struct.unpack(">H", pdu[1:3])[0]
            result["quantity"] = struct.unpack(">H", pdu[3:5])[0]
    return result


class ModbusBaseline:
    """Maintains baseline statistics for Modbus communication patterns."""

    def __init__(self):
        self.function_code_counts = defaultdict(lambda: defaultdict(int))
        self.register_ranges = defaultdict(set)
        self.timing_windows = defaultdict(list)
        self.register_values = defaultdict(list)
        self.total_packets = defaultdict(int)

    def record(self, src_ip, dst_ip, function_code, timestamp, registers=None, values=None):
        pair_key = (src_ip, dst_ip)
        self.function_code_counts[pair_key][function_code] += 1
        self.total_packets[pair_key] += 1
        self.timing_windows[pair_key].append(timestamp)
        if registers:
            for reg in registers:
                self.register_ranges[pair_key].add(reg)
        if values and registers:
            for reg, val in zip(registers, values):
                self.register_values[reg].append(val)

    def get_fc_distribution(self, src_ip, dst_ip):
        pair_key = (src_ip, dst_ip)
        total = self.total_packets[pair_key]
        if total == 0:
            return {}
        return {
            fc: count / total
            for fc, count in self.function_code_counts[pair_key].items()
        }

    def get_timing_stats(self, src_ip, dst_ip):
        pair_key = (src_ip, dst_ip)
        timestamps = self.timing_windows[pair_key]
        if len(timestamps) < 10:
            return None
        intervals = np.diff(sorted(timestamps))
        return {"mean": float(np.mean(intervals)), "std": float(np.std(intervals))}

    def get_register_stats(self, register_addr):
        values = self.register_values.get(register_addr, [])
        if len(values) < 5:
            return None
        return {
            "min": float(np.min(values)),
            "max": float(np.max(values)),
            "mean": float(np.mean(values)),
            "std": float(np.std(values)),
        }

    def save(self, filepath):
        data = {
            "fc_counts": {
                f"{k[0]}->{k[1]}": dict(v)
                for k, v in self.function_code_counts.items()
            },
            "register_ranges": {
                f"{k[0]}->{k[1]}": sorted(v)
                for k, v in self.register_ranges.items()
            },
            "total_packets": {
                f"{k[0]}->{k[1]}": v for k, v in self.total_packets.items()
            },
            "register_values": {
                str(k): {
                    "min": float(np.min(v)),
                    "max": float(np.max(v)),
                    "mean": float(np.mean(v)),
                    "std": float(np.std(v)),
                    "count": len(v),
                }
                for k, v in self.register_values.items()
                if len(v) >= 5
            },
        }
        Path(filepath).write_text(json.dumps(data, indent=2))
        logger.info("Baseline saved to %s", filepath)

    def load(self, filepath):
        data = json.loads(Path(filepath).read_text())
        for pair_str, fc_dict in data.get("fc_counts", {}).items():
            src, dst = pair_str.split("->")
            for fc_str, count in fc_dict.items():
                self.function_code_counts[(src, dst)][int(fc_str)] = count
        for pair_str, regs in data.get("register_ranges", {}).items():
            src, dst = pair_str.split("->")
            self.register_ranges[(src, dst)] = set(regs)
        for pair_str, total in data.get("total_packets", {}).items():
            src, dst = pair_str.split("->")
            self.total_packets[(src, dst)] = total
        logger.info("Baseline loaded from %s", filepath)


class ModbusAnomalyDetector:
    """Detects anomalies in Modbus TCP traffic based on baseline profiles."""

    def __init__(self, authorized_masters=None, authorized_writers=None,
                 register_limits=None, baseline=None):
        self.authorized_masters = set(authorized_masters or [])
        self.authorized_writers = set(authorized_writers or [])
        self.register_limits = register_limits or {}
        self.baseline = baseline or ModbusBaseline()
        self.alerts = []
        self.previous_register_values = {}
        self.exception_counts = defaultdict(lambda: defaultdict(int))
        self.device_scan_tracker = defaultdict(set)
        self.connection_counts = defaultdict(list)

    def analyze_packet(self, src_ip, dst_ip, dst_port, raw_payload, timestamp):
        """Analyze a single Modbus TCP packet for anomalies."""
        if dst_port != MODBUS_PORT:
            return []
        packet_alerts = []

        rogue = self._check_rogue_master(src_ip, dst_ip, timestamp)
        if rogue:
            packet_alerts.append(rogue)

        mbap = parse_mbap_header(raw_payload)
        if not mbap:
            packet_alerts.append({
                "alert": "MALFORMED_MBAP_HEADER",
                "severity": "MEDIUM",
                "timestamp": timestamp,
                "src_ip": src_ip,
                "dst_ip": dst_ip,
                "description": "Modbus frame with invalid MBAP header",
            })
            return packet_alerts

        pdu = parse_modbus_pdu(raw_payload)
        if not pdu:
            return packet_alerts

        fc = pdu["function_code"]

        if pdu["is_exception"]:
            exc_alerts = self._check_exception_burst(
                src_ip, dst_ip, fc, pdu.get("exception_code"), timestamp
            )
            if exc_alerts:
                packet_alerts.extend(exc_alerts)
            return packet_alerts

        unauth_write = self._check_unauthorized_write(src_ip, fc, dst_ip, timestamp)
        if unauth_write:
            packet_alerts.append(unauth_write)

        recon = self._check_reconnaissance(src_ip, dst_ip, fc, timestamp)
        if recon:
            packet_alerts.append(recon)

        fc_anomaly = self._check_fc_anomaly(src_ip, dst_ip, fc, timestamp)
        if fc_anomaly:
            packet_alerts.append(fc_anomaly)

        reg_alerts = self._check_register_values(src_ip, dst_ip, pdu, timestamp)
        if reg_alerts:
            packet_alerts.extend(reg_alerts)

        timing = self._check_timing_anomaly(src_ip, dst_ip, timestamp)
        if timing:
            packet_alerts.append(timing)

        registers = self._extract_registers(pdu)
        values = self._extract_values(pdu)
        self.baseline.record(src_ip, dst_ip, fc, timestamp, registers, values)

        self.alerts.extend(packet_alerts)
        return packet_alerts

    def _check_rogue_master(self, src_ip, dst_ip, timestamp):
        if self.authorized_masters and src_ip not in self.authorized_masters:
            return {
                "alert": "ROGUE_MODBUS_MASTER",
                "severity": "CRITICAL",
                "timestamp": timestamp,
                "src_ip": src_ip,
                "target_slave": dst_ip,
                "description": f"Unauthorized device {src_ip} initiating Modbus connection "
                               f"to slave {dst_ip}",
            }
        return None

    def _check_unauthorized_write(self, src_ip, function_code, dst_ip, timestamp):
        if function_code in WRITE_FUNCTION_CODES and self.authorized_writers and \
                src_ip not in self.authorized_writers:
            return {
                "alert": "UNAUTHORIZED_MODBUS_WRITE",
                "severity": "CRITICAL",
                "timestamp": timestamp,
                "src_ip": src_ip,
                "dst_ip": dst_ip,
                "function_code": function_code,
                "function_name": FUNCTION_CODE_NAMES.get(function_code, "Unknown"),
                "description": f"Write FC {function_code} "
                               f"({FUNCTION_CODE_NAMES.get(function_code, 'Unknown')}) "
                               f"from unauthorized source {src_ip}",
            }
        return None

    def _check_reconnaissance(self, src_ip, dst_ip, function_code, timestamp):
        if function_code in DIAGNOSTIC_FUNCTION_CODES:
            self.device_scan_tracker[src_ip].add(dst_ip)
            targets = self.device_scan_tracker[src_ip]
            severity = "CRITICAL" if len(targets) >= 5 else "HIGH"
            return {
                "alert": "DEVICE_ENUMERATION",
                "severity": severity,
                "timestamp": timestamp,
                "src_ip": src_ip,
                "dst_ip": dst_ip,
                "function_code": function_code,
                "function_name": FUNCTION_CODE_NAMES.get(function_code, "Unknown"),
                "unique_targets": len(targets),
                "description": f"Diagnostic FC {function_code} from {src_ip} to {dst_ip}. "
                               f"Total unique targets scanned: {len(targets)}",
            }
        return None

    def _check_fc_anomaly(self, src_ip, dst_ip, function_code, timestamp):
        pair_key = (src_ip, dst_ip)
        baseline_dist = self.baseline.get_fc_distribution(src_ip, dst_ip)
        if not baseline_dist:
            return None
        if function_code not in self.baseline.function_code_counts.get(pair_key, {}):
            return {
                "alert": "NEW_FUNCTION_CODE",
                "severity": "HIGH",
                "timestamp": timestamp,
                "src_ip": src_ip,
                "dst_ip": dst_ip,
                "function_code": function_code,
                "function_name": FUNCTION_CODE_NAMES.get(function_code, "Unknown"),
                "description": f"FC {function_code} "
                               f"({FUNCTION_CODE_NAMES.get(function_code, 'Unknown')}) "
                               f"never seen before for {src_ip} -> {dst_ip}",
            }
        return None

    def _check_exception_burst(self, src_ip, dst_ip, function_code, exception_code, timestamp):
        pair_key = (src_ip, dst_ip)
        minute_key = int(timestamp // 60)
        self.exception_counts[pair_key][minute_key] += 1
        count = self.exception_counts[pair_key][minute_key]
        exception_names = {1: "Illegal Function", 2: "Illegal Data Address",
                           3: "Illegal Data Value", 4: "Slave Device Failure"}
        if count == 10:
            return [{
                "alert": "EXCEPTION_BURST",
                "severity": "HIGH",
                "timestamp": timestamp,
                "src_ip": src_ip,
                "dst_ip": dst_ip,
                "exception_code": exception_code,
                "exception_name": exception_names.get(exception_code, "Unknown"),
                "count_per_minute": count,
                "description": f"Burst of {count} Modbus exceptions from {dst_ip} in response "
                               f"to requests from {src_ip}. Possible scanning or fuzzing.",
            }]
        return None

    def _check_register_values(self, src_ip, dst_ip, pdu, timestamp):
        alerts = []
        fc = pdu["function_code"]
        if fc == 6:
            reg = pdu.get("register_address")
            val = pdu.get("register_value")
            if reg is not None and val is not None:
                alert = self._validate_register(reg, val, src_ip, dst_ip, timestamp)
                if alert:
                    alerts.extend(alert)
        elif fc == 16:
            start = pdu.get("start_address", 0)
            values = pdu.get("values", [])
            for i, val in enumerate(values):
                reg = start + i
                alert = self._validate_register(reg, val, src_ip, dst_ip, timestamp)
                if alert:
                    alerts.extend(alert)
        return alerts

    def _validate_register(self, register_addr, new_value, src_ip, dst_ip, timestamp):
        alerts = []
        if register_addr in self.register_limits:
            limits = self.register_limits[register_addr]
            if new_value < limits.get("min", float("-inf")) or \
                    new_value > limits.get("max", float("inf")):
                alerts.append({
                    "alert": "REGISTER_VALUE_OUT_OF_RANGE",
                    "severity": "CRITICAL",
                    "timestamp": timestamp,
                    "src_ip": src_ip,
                    "dst_ip": dst_ip,
                    "register": register_addr,
                    "register_name": limits.get("name", f"Register {register_addr}"),
                    "value": new_value,
                    "safe_range": f"{limits.get('min', 'N/A')}-{limits.get('max', 'N/A')} "
                                  f"{limits.get('unit', '')}",
                    "description": f"Register {register_addr} "
                                   f"({limits.get('name', 'Unknown')}) set to {new_value}, "
                                   f"outside safe range "
                                   f"{limits.get('min')}-{limits.get('max')} "
                                   f"{limits.get('unit', '')}",
                })
        prev = self.previous_register_values.get(register_addr)
        if prev is not None and register_addr in self.register_limits:
            limits = self.register_limits[register_addr]
            max_rate = limits.get("max_rate")
            if max_rate and abs(new_value - prev) > max_rate:
                alerts.append({
                    "alert": "REGISTER_VALUE_EXCESSIVE_RATE",
                    "severity": "HIGH",
                    "timestamp": timestamp,
                    "src_ip": src_ip,
                    "dst_ip": dst_ip,
                    "register": register_addr,
                    "register_name": limits.get("name", f"Register {register_addr}"),
                    "previous_value": prev,
                    "new_value": new_value,
                    "change": abs(new_value - prev),
                    "max_allowed_change": max_rate,
                    "description": f"Register {register_addr} changed by "
                                   f"{abs(new_value - prev)} (max allowed: {max_rate})",
                })
        self.previous_register_values[register_addr] = new_value
        return alerts if alerts else None

    def _check_timing_anomaly(self, src_ip, dst_ip, timestamp):
        stats = self.baseline.get_timing_stats(src_ip, dst_ip)
        if not stats or stats["std"] == 0:
            return None
        pair_key = (src_ip, dst_ip)
        timestamps = self.baseline.timing_windows[pair_key]
        if len(timestamps) < 2:
            return None
        interval = timestamp - timestamps[-1]
        deviation = abs(interval - stats["mean"]) / stats["std"]
        if deviation > 3.0:
            return {
                "alert": "TIMING_ANOMALY",
                "severity": "MEDIUM",
                "timestamp": timestamp,
                "src_ip": src_ip,
                "dst_ip": dst_ip,
                "interval_seconds": round(interval, 4),
                "expected_mean": round(stats["mean"], 4),
                "expected_std": round(stats["std"], 4),
                "deviation_sigma": round(deviation, 2),
                "description": f"Inter-packet interval {interval:.4f}s deviates "
                               f"{deviation:.1f} sigma from mean {stats['mean']:.4f}s",
            }
        return None

    def _extract_registers(self, pdu):
        fc = pdu["function_code"]
        if fc in (1, 2, 3, 4):
            start = pdu.get("start_address", 0)
            qty = pdu.get("quantity", 0)
            return list(range(start, start + qty))
        elif fc == 5:
            addr = pdu.get("coil_address")
            return [addr] if addr is not None else []
        elif fc == 6:
            addr = pdu.get("register_address")
            return [addr] if addr is not None else []
        elif fc in (15, 16):
            start = pdu.get("start_address", 0)
            qty = pdu.get("quantity", 0)
            return list(range(start, start + qty))
        return []

    def _extract_values(self, pdu):
        fc = pdu["function_code"]
        if fc == 5:
            val = pdu.get("coil_value")
            return [val] if val is not None else []
        elif fc == 6:
            val = pdu.get("register_value")
            return [val] if val is not None else []
        elif fc == 16:
            return pdu.get("values", [])
        return []

    def generate_report(self):
        """Generate JSON anomaly report."""
        severity_order = {"CRITICAL": 0, "HIGH": 1, "MEDIUM": 2, "LOW": 3}
        sorted_alerts = sorted(
            self.alerts, key=lambda a: severity_order.get(a.get("severity", "LOW"), 99)
        )
        report = {
            "report_generated": datetime.now(timezone.utc).isoformat(),
            "total_anomalies": len(sorted_alerts),
            "severity_summary": {
                "CRITICAL": sum(1 for a in sorted_alerts if a["severity"] == "CRITICAL"),
                "HIGH": sum(1 for a in sorted_alerts if a["severity"] == "HIGH"),
                "MEDIUM": sum(1 for a in sorted_alerts if a["severity"] == "MEDIUM"),
                "LOW": sum(1 for a in sorted_alerts if a["severity"] == "LOW"),
            },
            "alerts": sorted_alerts,
        }
        return report


def analyze_pcap(pcap_file, detector):
    """Analyze a pcap file for Modbus TCP anomalies using Scapy."""
    try:
        from scapy.all import rdpcap, TCP, IP
    except ImportError:
        logger.error("Scapy is required for pcap analysis: pip install scapy")
        return

    logger.info("Loading pcap: %s", pcap_file)
    packets = rdpcap(pcap_file)
    modbus_count = 0

    for pkt in packets:
        if not pkt.haslayer(TCP) or not pkt.haslayer(IP):
            continue
        tcp = pkt[TCP]
        ip = pkt[IP]
        if tcp.dport != MODBUS_PORT and tcp.sport != MODBUS_PORT:
            continue
        payload = bytes(tcp.payload)
        if len(payload) < MBAP_HEADER_SIZE + 1:
            continue

        dst_port = tcp.dport
        src_ip = ip.src
        dst_ip = ip.dst
        timestamp = float(pkt.time)

        alerts = detector.analyze_packet(src_ip, dst_ip, dst_port, payload, timestamp)
        modbus_count += 1
        for alert in alerts:
            logger.warning("[%s] %s: %s", alert["severity"], alert["alert"],
                           alert["description"])

    logger.info("Analyzed %d Modbus packets from %d total packets", modbus_count, len(packets))


def live_capture(interface, detector, duration=0):
    """Capture and analyze Modbus TCP traffic in real-time using Scapy."""
    try:
        from scapy.all import sniff, TCP, IP
    except ImportError:
        logger.error("Scapy is required for live capture: pip install scapy")
        return

    def process_packet(pkt):
        if not pkt.haslayer(TCP) or not pkt.haslayer(IP):
            return
        tcp = pkt[TCP]
        ip = pkt[IP]
        payload = bytes(tcp.payload)
        if len(payload) < MBAP_HEADER_SIZE + 1:
            return
        alerts = detector.analyze_packet(
            ip.src, ip.dst, tcp.dport, payload, float(pkt.time)
        )
        for alert in alerts:
            logger.warning("[%s] %s: %s", alert["severity"], alert["alert"],
                           alert["description"])

    logger.info("Starting live capture on %s (filter: port 502)", interface)
    kwargs = {"iface": interface, "filter": "tcp port 502", "prn": process_packet,
              "store": False}
    if duration > 0:
        kwargs["timeout"] = duration
    sniff(**kwargs)


def main():
    parser = argparse.ArgumentParser(
        description="Modbus TCP Traffic Anomaly Detector for SCADA/ICS Networks"
    )
    parser.add_argument("--pcap", help="Path to pcap file to analyze")
    parser.add_argument("--interface", help="Network interface for live capture")
    parser.add_argument("--duration", type=int, default=0,
                        help="Live capture duration in seconds (0=indefinite)")
    parser.add_argument("--authorized-masters", nargs="+",
                        help="List of authorized Modbus master IPs")
    parser.add_argument("--authorized-writers", nargs="+",
                        help="List of IPs authorized to send write commands")
    parser.add_argument("--register-limits-file",
                        help="JSON file defining safe register value ranges")
    parser.add_argument("--baseline-file",
                        help="Path to load/save baseline profile")
    parser.add_argument("--baseline-mode", action="store_true",
                        help="Run in baseline-building mode (no alerting)")
    parser.add_argument("--output", default="modbus_anomaly_report.json",
                        help="Output report file (default: modbus_anomaly_report.json)")
    args = parser.parse_args()

    register_limits = {}
    if args.register_limits_file:
        register_limits = json.loads(Path(args.register_limits_file).read_text())
        logger.info("Loaded register limits for %d registers", len(register_limits))

    baseline = ModbusBaseline()
    if args.baseline_file and Path(args.baseline_file).exists() and not args.baseline_mode:
        baseline.load(args.baseline_file)

    detector = ModbusAnomalyDetector(
        authorized_masters=args.authorized_masters,
        authorized_writers=args.authorized_writers,
        register_limits=register_limits,
        baseline=baseline,
    )

    if args.pcap:
        analyze_pcap(args.pcap, detector)
    elif args.interface:
        live_capture(args.interface, detector, args.duration)
    else:
        parser.error("Either --pcap or --interface is required")

    if args.baseline_mode and args.baseline_file:
        baseline.save(args.baseline_file)
        logger.info("Baseline mode complete. Profile saved to %s", args.baseline_file)
    else:
        report = detector.generate_report()
        Path(args.output).write_text(json.dumps(report, indent=2, default=str))
        logger.info("Report saved to %s (%d anomalies detected)",
                     args.output, report["total_anomalies"])


if __name__ == "__main__":
    main()

===== END FILE =====
