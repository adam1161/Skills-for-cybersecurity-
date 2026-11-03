

===== FILE: skills/performing-purple-team-atomic-testing/SKILL.md =====
---
name: performing-purple-team-atomic-testing
description: 'Executes Atomic Red Team tests mapped to MITRE ATT&CK via Invoke-AtomicRedTeam
  PowerShell, generates ATT&CK Navigator coverage heatmaps, correlates results against
  Sigma rules, and runs detection validation loops to measure blue team visibility.
  Use for purple team exercises, atomic test execution, ATT&CK coverage assessment,
  or detection engineering validation.

  '
domain: cybersecurity
subdomain: purple-team
tags:
- purple-team
- atomic-red-team
- mitre-attack
- detection-engineering
- adversary-emulation
version: 1.0.0
author: mukul975
license: Apache-2.0
nist_ai_rmf:
- MEASURE-2.7
- MAP-5.1
- MANAGE-2.4
atlas_techniques:
- AML.T0070
- AML.T0066
- AML.T0082
d3fend_techniques:
- Executable Denylisting
- Execution Isolation
- File Metadata Consistency Validation
- Content Format Conversion
- File Content Analysis
nist_csf:
- ID.RA-01
- DE.AE-07
- GV.OV-02
mitre_attack:
- T1078
- T1190
- T1059
---

# Performing Purple Team Atomic Testing

## When to Use

- Validating detection coverage against specific MITRE ATT&CK techniques
- Running purple team exercises using Atomic Red Team test library
- Performing ATT&CK coverage gap analysis to identify blind spots in SIEM/EDR
- Building a detection validation loop: execute atomic test, check SIEM, tune rule, retest
- Generating ATT&CK Navigator heatmap layers for executive reporting
- Automating continuous atomic testing in CI/CD or scheduled pipelines
- Mapping threat intelligence reports to executable atomic tests

**Do not use** for full-scope red team engagements requiring custom implants or live adversary simulation beyond atomic tests; use Caldera, SCYTHE, or Cobalt Strike for advanced adversary emulation.

**DISCLAIMER**: Atomic Red Team tests execute real attack techniques. Run only on systems you own or have explicit written authorization to test. Many tests modify system state, create artifacts, or trigger security alerts. Always execute cleanup commands after testing. Never run atomic tests in production without risk acceptance from stakeholders.

## Prerequisites

- Windows host with PowerShell 5.1+ or PowerShell Core 7+ (Linux/macOS supported for cross-platform atomics)
- Invoke-AtomicRedTeam PowerShell module installed from PSGallery
- Atomic Red Team atomics repository cloned locally
- SIEM/EDR with log ingestion from test endpoints (Splunk, Elastic, Microsoft Sentinel, CrowdStrike)
- MITRE ATT&CK Navigator (web-based or local instance) for layer visualization
- Python 3.9+ with `mitreattack-python`, `pyyaml`, and `requests` for automation scripts
- Sigma rules repository for detection correlation
- Administrative/root access on test endpoints
- Isolated test environment (lab, sandbox, or dedicated test range)

## Workflow

### Step 1: Install and Configure Invoke-AtomicRedTeam

Set up the execution framework and download the atomics library:

```powershell
# Install the PowerShell execution module
Install-Module -Name invoke-atomicredteam -Scope CurrentUser -Force
Install-Module -Name powershell-yaml -Scope CurrentUser -Force

# Import the module
Import-Module invoke-atomicredteam

# Install atomics to default location (C:\AtomicRedTeam\atomics)
IEX (IEX (New-Object System.Net.WebClient).DownloadString(
    'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1'
)); Install-AtomicRedTeam -getAtomics -Force

# Verify installation - list available techniques
$atomicsPath = "C:\AtomicRedTeam\atomics"
$techniques = Get-ChildItem $atomicsPath -Directory | Where-Object { $_.Name -match '^T\d{4}' }
Write-Host "Available techniques: $($techniques.Count)"

# Configure execution logging
$env:ARTLOG = "C:\AtomicRedTeam\logs"
if (-not (Test-Path $env:ARTLOG)) { New-Item -Path $env:ARTLOG -ItemType Directory }
```

### Step 2: Enumerate and Select Atomic Tests

Inventory available tests and select targets based on threat intelligence or gap analysis:

```powershell
# List all tests for a specific technique
Invoke-AtomicTest T1059.001 -ShowDetailsBrief

# Show full details including attack commands and cleanup
Invoke-AtomicTest T1059.001 -ShowDetails

# List tests for a tactic (e.g., Persistence)
$persistenceTechniques = @(
    "T1547.001",  # Boot or Logon Autostart - Registry Run Keys
    "T1053.005",  # Scheduled Task
    "T1136.001",  # Create Account - Local Account
    "T1543.003",  # Create or Modify System Process - Windows Service
    "T1546.001",  # Event Triggered Execution - Change Default File Association
    "T1574.001",  # Hijack Execution Flow - DLL Search Order Hijacking
    "T1197"       # BITS Jobs
)

foreach ($tech in $persistenceTechniques) {
    Write-Host "`n=== $tech ===" -ForegroundColor Cyan
    try {
        Invoke-AtomicTest $tech -ShowDetailsBrief
    } catch {
        Write-Host "  No tests available" -ForegroundColor Yellow
    }
}

# Get all atomic techniques from YAML files programmatically
$allAtomics = Get-ChildItem "$atomicsPath\T*\T*.yaml" -Recurse |
    ForEach-Object {
        $yaml = Get-Content $_.FullName -Raw | ConvertFrom-Yaml
        [PSCustomObject]@{
            TechniqueId   = $yaml.attack_technique
            TechniqueName = $yaml.display_name
            TestCount     = $yaml.atomic_tests.Count
            Platforms     = ($yaml.atomic_tests.supported_platforms | Sort-Object -Unique) -join ", "
        }
    }

$allAtomics | Sort-Object TechniqueId | Format-Table -AutoSize
Write-Host "Total techniques with tests: $($allAtomics.Count)"
Write-Host "Total individual tests: $(($allAtomics | Measure-Object -Property TestCount -Sum).Sum)"
```

### Step 3: Execute Atomic Tests with Logging

Run tests with pre/post logging for detection validation:

```powershell
# Execute a single test by technique ID (runs all tests for that technique)
Invoke-AtomicTest T1059.001

# Execute a specific test by number
Invoke-AtomicTest T1059.001 -TestNumbers 1

# Execute by test name
Invoke-AtomicTest T1059.001 -TestNames "Mimikatz - Cradled Invoke Expression"

# Execute by GUID
Invoke-AtomicTest T1059.001 -TestGuids "2e803f96-4e33-4c2c-b0c8-1c10cbb3945f"

# Execute with prerequisite check and installation
Invoke-AtomicTest T1059.001 -TestNumbers 1 -CheckPrereqs
Invoke-AtomicTest T1059.001 -TestNumbers 1 -GetPrereqs
Invoke-AtomicTest T1059.001 -TestNumbers 1

# Execute with timeout (seconds)
Invoke-AtomicTest T1003.001 -TimeoutSeconds 120

# Cleanup after testing
Invoke-AtomicTest T1059.001 -TestNumbers 1 -Cleanup

# Execute with full logging wrapper
function Invoke-AtomicWithLogging {
    param(
        [string]$TechniqueId,
        [int[]]$TestNumbers,
        [string]$LogPath = "C:\AtomicRedTeam\logs"
    )

    $timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
    $logFile = Join-Path $LogPath "${TechniqueId}_${timestamp}.json"

    $result = @{
        technique_id  = $TechniqueId
        test_numbers  = $TestNumbers
        start_time    = (Get-Date).ToString("o")
        hostname      = $env:COMPUTERNAME
        username      = $env:USERNAME
        results       = @()
    }

    foreach ($testNum in $TestNumbers) {
        $testResult = @{
            test_number = $testNum
            status      = "unknown"
            start_time  = (Get-Date).ToString("o")
        }

        try {
            # Show what will execute
            $details = Invoke-AtomicTest $TechniqueId -TestNumbers $testNum -ShowDetails 2>&1
            $testResult["details"] = $details | Out-String

            # Execute the test
            Invoke-AtomicTest $TechniqueId -TestNumbers $testNum -Confirm:$false
            $testResult["status"] = "executed"
        } catch {
            $testResult["status"] = "failed"
            $testResult["error"] = $_.Exception.Message
        }

        $testResult["end_time"] = (Get-Date).ToString("o")
        $result.results += $testResult

        # Wait for SIEM ingestion
        Start-Sleep -Seconds 30
    }

    $result["end_time"] = (Get-Date).ToString("o")
    $result | ConvertTo-Json -Depth 10 | Set-Content $logFile
    Write-Host "Log written to: $logFile" -ForegroundColor Green
    return $result
}

# Usage
Invoke-AtomicWithLogging -TechniqueId "T1059.001" -TestNumbers @(1, 2, 3)
```

### Step 4: Validate Detections in SIEM

Query your SIEM to confirm whether atomic tests generated alerts:

```
Splunk SPL Queries for Detection Validation:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

-- T1059.001: PowerShell Execution
index=windows sourcetype="WinEventLog:Microsoft-Windows-PowerShell/Operational"
  EventCode=4104
  | eval script_block=ScriptBlockText
  | where len(script_block) > 500
  | stats count by host, script_block
  | sort -count

-- T1003.001: LSASS Memory Credential Dumping
index=windows sourcetype="WinEventLog:Security" EventCode=4663
  ObjectName="*lsass*"
  | stats count by host, SubjectUserName, ProcessName
  | where count > 0

-- T1547.001: Registry Run Key Persistence
index=windows sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational"
  EventCode=13
  TargetObject="*\\CurrentVersion\\Run*"
  | stats count by host, Image, TargetObject, Details

-- T1053.005: Scheduled Task Creation
index=windows sourcetype="WinEventLog:Security" EventCode=4698
  | stats count by host, SubjectUserName, TaskName, TaskContent
  | sort -count

-- Generic: Hunt for Atomic Red Team artifacts
index=windows (sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational"
  OR sourcetype="WinEventLog:Security")
  | search "*AtomicRedTeam*" OR "*atomic*" OR "*Invoke-AtomicTest*"
  | stats count by sourcetype, EventCode, host
```

```
Elastic / KQL Queries:
━━━━━━━━━━━━━━━━━━━━━

-- PowerShell script block logging
event.code: "4104" and powershell.file.script_block_text: *

-- Sysmon process creation from AtomicRedTeam paths
event.code: "1" and process.executable: *AtomicRedTeam*

-- Registry modification (persistence)
event.code: "13" and registry.path: *CurrentVersion\\Run*

-- Credential access indicators
event.code: "10" and winlog.event_data.TargetImage: *lsass.exe*
```

### Step 5: ATT&CK Coverage Gap Analysis

Generate a coverage matrix comparing tested vs. detected techniques:

```python
#!/usr/bin/env python3
"""ATT&CK coverage gap analysis - compares atomic test results against SIEM detections."""

import json
import os
import yaml
from pathlib import Path
from datetime import datetime


def load_atomics_inventory(atomics_path):
    """Parse all atomic test YAML files to build technique inventory."""
    inventory = {}
    atomics_dir = Path(atomics_path)

    for yaml_file in atomics_dir.glob("T*/T*.yaml"):
        try:
            with open(yaml_file, "r", encoding="utf-8") as f:
                data = yaml.safe_load(f)

            tech_id = data.get("attack_technique", "")
            if not tech_id:
                continue

            tests = data.get("atomic_tests", [])
            inventory[tech_id] = {
                "name": data.get("display_name", "Unknown"),
                "test_count": len(tests),
                "platforms": list(set(
                    p for t in tests
                    for p in t.get("supported_platforms", [])
                )),
                "tests": [
                    {
                        "name": t.get("name", "Unnamed"),
                        "description": t.get("description", ""),
                        "platforms": t.get("supported_platforms", []),
                        "executor": t.get("executor", {}).get("name", "unknown"),
                    }
                    for t in tests
                ],
            }
        except Exception as e:
            print(f"[WARN] Failed to parse {yaml_file}: {e}")

    return inventory


def load_execution_logs(log_dir):
    """Load atomic test execution logs."""
    executed = {}
    log_path = Path(log_dir)

    if not log_path.exists():
        return executed

    for log_file in log_path.glob("T*_*.json"):
        try:
            with open(log_file, "r") as f:
                data = json.load(f)
            tech_id = data.get("technique_id", "")
            if tech_id:
                if tech_id not in executed:
                    executed[tech_id] = {
                        "executions": [],
                        "last_executed": data.get("end_time", ""),
                    }
                executed[tech_id]["executions"].append({
                    "timestamp": data.get("start_time", ""),
                    "results": data.get("results", []),
                })
        except Exception as e:
            print(f"[WARN] Failed to parse {log_file}: {e}")

    return executed


def load_detection_results(detection_file):
    """Load SIEM detection validation results (JSON export from SIEM queries)."""
    if not os.path.exists(detection_file):
        return {}

    with open(detection_file, "r") as f:
        data = json.load(f)

    detections = {}
    for entry in data:
        tech_id = entry.get("technique_id", "")
        if tech_id:
            detections[tech_id] = {
                "detected": entry.get("detected", False),
                "alert_count": entry.get("alert_count", 0),
                "rule_name": entry.get("rule_name", ""),
                "confidence": entry.get("confidence", "unknown"),
                "data_sources": entry.get("data_sources", []),
            }

    return detections


# MITRE ATT&CK tactic ordering for structured output
TACTIC_ORDER = [
    "reconnaissance", "resource-development", "initial-access",
    "execution", "persistence", "privilege-escalation",
    "defense-evasion", "credential-access", "discovery",
    "lateral-movement", "collection", "command-and-control",
    "exfiltration", "impact",
]

# Tactic-to-technique mapping for common techniques (subset for illustration)
TACTIC_TECHNIQUE_MAP = {
    "execution": [
        "T1059", "T1059.001", "T1059.003", "T1059.004", "T1059.005",
        "T1059.006", "T1059.007", "T1047", "T1053", "T1053.005",
        "T1129", "T1203", "T1569", "T1569.002",
    ],
    "persistence": [
        "T1547", "T1547.001", "T1547.004", "T1547.009",
        "T1053.005", "T1136", "T1136.001", "T1543", "T1543.003",
        "T1546", "T1546.001", "T1546.003", "T1574", "T1574.001",
        "T1197", "T1505", "T1505.003",
    ],
    "credential-access": [
        "T1003", "T1003.001", "T1003.002", "T1003.003",
        "T1003.004", "T1003.005", "T1003.006",
        "T1110", "T1110.001", "T1110.003",
        "T1555", "T1555.003", "T1552", "T1552.001",
        "T1558", "T1558.003",
    ],
    "defense-evasion": [
        "T1070", "T1070.001", "T1070.004",
        "T1218", "T1218.001", "T1218.003", "T1218.005",
        "T1218.010", "T1218.011",
        "T1027", "T1140", "T1562", "T1562.001",
        "T1036", "T1036.005",
    ],
    "discovery": [
        "T1082", "T1083", "T1087", "T1087.001", "T1087.002",
        "T1016", "T1049", "T1057", "T1069", "T1069.001",
        "T1069.002", "T1518", "T1518.001",
    ],
    "lateral-movement": [
        "T1021", "T1021.001", "T1021.002", "T1021.003",
        "T1021.004", "T1021.006", "T1570",
    ],
    "command-and-control": [
        "T1071", "T1071.001", "T1071.004",
        "T1105", "T1132", "T1573", "T1573.001",
        "T1219", "T1090",
    ],
    "exfiltration": [
        "T1041", "T1048", "T1048.003", "T1567",
    ],
    "impact": [
        "T1485", "T1486", "T1489", "T1490", "T1491",
    ],
}


def generate_coverage_report(atomics_inventory, execution_logs, detection_results):
    """Generate comprehensive coverage gap analysis."""
    report = {
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "summary": {},
        "tactics": {},
        "gaps": [],
        "recommendations": [],
    }

    total_available = len(atomics_inventory)
    total_executed = len(execution_logs)
    total_detected = sum(1 for d in detection_results.values() if d.get("detected"))

    report["summary"] = {
        "total_techniques_with_atomics": total_available,
        "total_techniques_executed": total_executed,
        "total_techniques_detected": total_detected,
        "execution_coverage_pct": round(
            (total_executed / total_available * 100) if total_available else 0, 1
        ),
        "detection_coverage_pct": round(
            (total_detected / total_executed * 100) if total_executed else 0, 1
        ),
    }

    # Per-tactic analysis
    for tactic, technique_ids in TACTIC_TECHNIQUE_MAP.items():
        tactic_data = {
            "techniques_available": 0,
            "techniques_executed": 0,
            "techniques_detected": 0,
            "gaps": [],
        }

        for tech_id in technique_ids:
            if tech_id in atomics_inventory:
                tactic_data["techniques_available"] += 1

                executed = tech_id in execution_logs
                detected = detection_results.get(tech_id, {}).get("detected", False)

                if executed:
                    tactic_data["techniques_executed"] += 1
                if detected:
                    tactic_data["techniques_detected"] += 1

                if executed and not detected:
                    gap = {
                        "technique_id": tech_id,
                        "technique_name": atomics_inventory[tech_id]["name"],
                        "tactic": tactic,
                        "status": "BLIND_SPOT",
                        "detail": "Test executed but no detection triggered",
                    }
                    tactic_data["gaps"].append(gap)
                    report["gaps"].append(gap)
                elif not executed and tech_id in atomics_inventory:
                    gap = {
                        "technique_id": tech_id,
                        "technique_name": atomics_inventory[tech_id]["name"],
                        "tactic": tactic,
                        "status": "NOT_TESTED",
                        "detail": "Atomic test available but not yet executed",
                    }
                    tactic_data["gaps"].append(gap)

        avail = tactic_data["techniques_available"]
        tactic_data["coverage_pct"] = round(
            (tactic_data["techniques_detected"] / avail * 100) if avail else 0, 1
        )
        report["tactics"][tactic] = tactic_data

    # Generate prioritized recommendations
    blind_spots = [g for g in report["gaps"] if g["status"] == "BLIND_SPOT"]
    if blind_spots:
        report["recommendations"].append({
            "priority": "CRITICAL",
            "action": f"Write detection rules for {len(blind_spots)} blind spot techniques",
            "techniques": [g["technique_id"] for g in blind_spots],
        })

    low_coverage_tactics = [
        t for t, d in report["tactics"].items() if d["coverage_pct"] < 30
    ]
    if low_coverage_tactics:
        report["recommendations"].append({
            "priority": "HIGH",
            "action": f"Expand testing in low-coverage tactics: {', '.join(low_coverage_tactics)}",
            "detail": "These tactics have less than 30% detection coverage",
        })

    return report


def generate_navigator_layer(atomics_inventory, execution_logs, detection_results,
                             layer_name="Purple Team Coverage"):
    """Generate ATT&CK Navigator layer JSON for heatmap visualization."""
    layer = {
        "name": layer_name,
        "versions": {
            "attack": "15",
            "navigator": "5.1",
            "layer": "4.5",
        },
        "domain": "enterprise-attack",
        "description": f"Purple team atomic testing coverage - Generated {datetime.utcnow().isoformat()}Z",
        "filters": {"platforms": ["Windows", "Linux", "macOS"]},
        "sorting": 0,
        "layout": {
            "layout": "side",
            "aggregateFunction": "average",
            "showID": True,
            "showName": True,
        },
        "hideDisabled": False,
        "techniques": [],
        "gradient": {
            "colors": ["#ff6666", "#ffeb3b", "#66bb6a"],
            "minValue": 0,
            "maxValue": 100,
        },
        "legendItems": [
            {"label": "No Coverage (Blind Spot)", "color": "#ff6666"},
            {"label": "Logged Only (Partial)", "color": "#ffeb3b"},
            {"label": "Alert/Detection Active", "color": "#66bb6a"},
            {"label": "Not Tested", "color": "#d3d3d3"},
        ],
        "metadata": [],
        "links": [],
        "showTacticRowBackground": True,
        "tacticRowBackground": "#dddddd",
        "selectTechniquesAcrossTactics": True,
        "selectSubtechniquesWithParent": False,
    }

    for tech_id, tech_data in atomics_inventory.items():
        executed = tech_id in execution_logs
        detection = detection_results.get(tech_id, {})
        detected = detection.get("detected", False)
        confidence = detection.get("confidence", "none")

        if detected and confidence in ("high", "medium"):
            score = 100
            color = "#66bb6a"  # Green - high confidence detection
            comment = f"DETECTED - {detection.get('rule_name', 'Alert active')}"
        elif detected:
            score = 50
            color = "#ffeb3b"  # Yellow - logged/partial
            comment = "PARTIAL - Detection exists but low confidence"
        elif executed:
            score = 0
            color = "#ff6666"  # Red - blind spot
            comment = "BLIND SPOT - Test executed, no detection"
        else:
            score = 0
            color = "#d3d3d3"  # Gray - not tested
            comment = f"NOT TESTED - {tech_data['test_count']} atomic tests available"

        technique_entry = {
            "techniqueID": tech_id,
            "tactic": "",
            "color": color,
            "comment": comment,
            "score": score,
            "enabled": True,
            "metadata": [
                {"name": "tests_available", "value": str(tech_data["test_count"])},
                {"name": "executed", "value": str(executed)},
                {"name": "detected", "value": str(detected)},
            ],
            "links": [],
            "showSubtechniques": False,
        }
        layer["techniques"].append(technique_entry)

    return layer


def print_coverage_report(report):
    """Print formatted coverage report to console."""
    print("=" * 72)
    print("PURPLE TEAM ATOMIC TESTING - COVERAGE GAP ANALYSIS")
    print("=" * 72)
    print(f"Generated: {report['generated_at']}")
    print()

    s = report["summary"]
    print("EXECUTIVE SUMMARY")
    print("-" * 40)
    print(f"  Techniques with atomics:  {s['total_techniques_with_atomics']}")
    print(f"  Techniques executed:      {s['total_techniques_executed']}")
    print(f"  Techniques detected:      {s['total_techniques_detected']}")
    print(f"  Execution coverage:       {s['execution_coverage_pct']}%")
    print(f"  Detection coverage:       {s['detection_coverage_pct']}%")
    print()

    print("PER-TACTIC COVERAGE")
    print("-" * 72)
    print(f"{'Tactic':<25} {'Available':>9} {'Executed':>9} {'Detected':>9} {'Coverage':>9}")
    print("-" * 72)
    for tactic in TACTIC_ORDER:
        if tactic in report["tactics"]:
            t = report["tactics"][tactic]
            bar = "#" * int(t["coverage_pct"] / 5) + "." * (20 - int(t["coverage_pct"] / 5))
            print(
                f"  {tactic:<23} {t['techniques_available']:>9} "
                f"{t['techniques_executed']:>9} {t['techniques_detected']:>9} "
                f"{t['coverage_pct']:>8.1f}%"
            )
    print()

    blind_spots = [g for g in report["gaps"] if g["status"] == "BLIND_SPOT"]
    if blind_spots:
        print("CRITICAL BLIND SPOTS (executed but not detected)")
        print("-" * 72)
        for gap in blind_spots:
            print(f"  [!] {gap['technique_id']} - {gap['technique_name']}")
            print(f"      Tactic: {gap['tactic']}")
        print()

    if report["recommendations"]:
        print("RECOMMENDATIONS")
        print("-" * 72)
        for rec in report["recommendations"]:
            print(f"  [{rec['priority']}] {rec['action']}")
            if "techniques" in rec:
                print(f"         Techniques: {', '.join(rec['techniques'][:10])}")
            if "detail" in rec:
                print(f"         {rec['detail']}")
        print()


if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser(description="ATT&CK coverage gap analysis for purple team testing")
    parser.add_argument("--atomics-path", default=r"C:\AtomicRedTeam\atomics",
                        help="Path to Atomic Red Team atomics directory")
    parser.add_argument("--log-dir", default=r"C:\AtomicRedTeam\logs",
                        help="Path to atomic test execution logs")
    parser.add_argument("--detections-file", default="detection_results.json",
                        help="Path to SIEM detection validation export (JSON)")
    parser.add_argument("--output-layer", default="navigator_layer.json",
                        help="Output path for ATT&CK Navigator layer JSON")
    parser.add_argument("--output-report", default="coverage_report.json",
                        help="Output path for coverage report JSON")
    args = parser.parse_args()

    print("[*] Loading atomics inventory...")
    inventory = load_atomics_inventory(args.atomics_path)
    print(f"    Found {len(inventory)} techniques with atomic tests")

    print("[*] Loading execution logs...")
    exec_logs = load_execution_logs(args.log_dir)
    print(f"    Found logs for {len(exec_logs)} techniques")

    print("[*] Loading detection results...")
    det_results = load_detection_results(args.detections_file)
    print(f"    Found detection data for {len(det_results)} techniques")

    print("[*] Generating coverage report...")
    report = generate_coverage_report(inventory, exec_logs, det_results)
    print_coverage_report(report)

    # Save report JSON
    with open(args.output_report, "w") as f:
        json.dump(report, f, indent=2)
    print(f"[+] Report saved to {args.output_report}")

    # Generate and save Navigator layer
    print("[*] Generating ATT&CK Navigator layer...")
    layer = generate_navigator_layer(inventory, exec_logs, det_results)
    with open(args.output_layer, "w") as f:
        json.dump(layer, f, indent=2)
    print(f"[+] Navigator layer saved to {args.output_layer}")
    print(f"    Import at: https://mitre-attack.github.io/attack-navigator/")
```

### Step 6: Run Continuous Atomic Testing Pipeline

Schedule recurring tests against priority techniques:

```powershell
# Continuous Atomic Testing - scheduled execution with validation
# Run as a scheduled task or via CI/CD pipeline

$PriorityTechniques = @(
    # Top MITRE ATT&CK techniques by prevalence (Red Canary Threat Detection Report)
    @{ Id = "T1059.001"; Name = "PowerShell" },
    @{ Id = "T1059.003"; Name = "Windows Command Shell" },
    @{ Id = "T1547.001"; Name = "Registry Run Keys" },
    @{ Id = "T1053.005"; Name = "Scheduled Task" },
    @{ Id = "T1003.001"; Name = "LSASS Memory" },
    @{ Id = "T1003.003"; Name = "NTDS" },
    @{ Id = "T1070.004"; Name = "File Deletion" },
    @{ Id = "T1218.011"; Name = "Rundll32" },
    @{ Id = "T1082";     Name = "System Information Discovery" },
    @{ Id = "T1105";     Name = "Ingress Tool Transfer" }
)

$ResultsLog = @()
$RunTimestamp = Get-Date -Format "yyyyMMdd_HHmmss"

foreach ($technique in $PriorityTechniques) {
    Write-Host "`n[*] Testing $($technique.Id) - $($technique.Name)" -ForegroundColor Cyan

    # Check prerequisites
    try {
        $prereqs = Invoke-AtomicTest $technique.Id -TestNumbers 1 -CheckPrereqs 2>&1
        $prereqsMet = $prereqs -notmatch "not met"
    } catch {
        $prereqsMet = $false
    }

    if (-not $prereqsMet) {
        Write-Host "    [!] Prerequisites not met, attempting install..." -ForegroundColor Yellow
        try {
            Invoke-AtomicTest $technique.Id -TestNumbers 1 -GetPrereqs 2>&1 | Out-Null
        } catch {
            Write-Host "    [!] Prereq install failed, skipping" -ForegroundColor Red
            $ResultsLog += [PSCustomObject]@{
                TechniqueId = $technique.Id
                Name        = $technique.Name
                Status      = "PREREQS_FAILED"
                Timestamp   = (Get-Date).ToString("o")
            }
            continue
        }
    }

    # Execute the test
    try {
        $startTime = Get-Date
        Invoke-AtomicTest $technique.Id -TestNumbers 1 -Confirm:$false
        $endTime = Get-Date
        $duration = ($endTime - $startTime).TotalSeconds

        Write-Host "    [+] Executed successfully (${duration}s)" -ForegroundColor Green

        $ResultsLog += [PSCustomObject]@{
            TechniqueId = $technique.Id
            Name        = $technique.Name
            Status      = "EXECUTED"
            Duration    = $duration
            Timestamp   = $startTime.ToString("o")
        }
    } catch {
        Write-Host "    [-] Execution failed: $($_.Exception.Message)" -ForegroundColor Red
        $ResultsLog += [PSCustomObject]@{
            TechniqueId = $technique.Id
            Name        = $technique.Name
            Status      = "FAILED"
            Error       = $_.Exception.Message
            Timestamp   = (Get-Date).ToString("o")
        }
    }

    # Cleanup
    try {
        Invoke-AtomicTest $technique.Id -TestNumbers 1 -Cleanup 2>&1 | Out-Null
        Write-Host "    [+] Cleanup completed" -ForegroundColor DarkGray
    } catch {
        Write-Host "    [!] Cleanup failed" -ForegroundColor Yellow
    }

    # Brief pause between tests for log ingestion
    Start-Sleep -Seconds 10
}

# Export results
$ResultsLog | Export-Csv "C:\AtomicRedTeam\logs\continuous_test_$RunTimestamp.csv" -NoTypeInformation
$ResultsLog | Format-Table -AutoSize

# Summary
$executed = ($ResultsLog | Where-Object Status -eq "EXECUTED").Count
$failed = ($ResultsLog | Where-Object Status -ne "EXECUTED").Count
Write-Host "`n=== CONTINUOUS TEST SUMMARY ===" -ForegroundColor Cyan
Write-Host "  Executed: $executed / $($PriorityTechniques.Count)" -ForegroundColor Green
Write-Host "  Failed:   $failed / $($PriorityTechniques.Count)" -ForegroundColor Red
```

### Step 7: Detection Validation Loop

Close the purple team loop by correlating tests with detections and iterating:

```
Detection Validation Loop Workflow:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ┌──────────────────┐
  │ 1. SELECT        │  Choose technique from threat intel or gap report
  │    TECHNIQUE     │  Map to Atomic Red Team test ID
  └───────┬──────────┘
          │
  ┌───────▼──────────┐
  │ 2. EXECUTE       │  Run Invoke-AtomicTest with logging
  │    ATOMIC TEST   │  Record timestamp, hostname, test details
  └───────┬──────────┘
          │
  ┌───────▼──────────┐
  │ 3. WAIT FOR      │  Allow 30-60 seconds for log ingestion
  │    INGESTION     │  Ensure Sysmon, WinEventLog, EDR agents forward data
  └───────┬──────────┘
          │
  ┌───────▼──────────┐
  │ 4. QUERY SIEM    │  Search for correlated alerts matching technique
  │    FOR ALERTS    │  Check: Did our detection rules fire?
  └───────┬──────────┘
          │
     ┌────┴────┐
     │DETECTED?│
     └────┬────┘
    YES   │   NO
  ┌───────▼──────────┐   ┌──────────────────┐
  │ 5a. MARK GREEN   │   │ 5b. WRITE NEW    │
  │  Update Navigator│   │  Sigma/SIEM Rule │
  │  layer, log pass │   │  for technique   │
  └───────┬──────────┘   └───────┬──────────┘
          │                       │
          │               ┌───────▼──────────┐
          │               │ 5c. DEPLOY RULE  │
          │               │  Push to SIEM    │
          │               └───────┬──────────┘
          │                       │
          │               ┌───────▼──────────┐
          │               │ 5d. RE-EXECUTE   │
          │               │  Re-run atomic   │
          │               │  test, validate  │
          │               └───────┬──────────┘
          │                       │
  ┌───────▼───────────────────────▼──┐
  │ 6. CLEANUP                       │
  │    Run Invoke-AtomicTest -Cleanup│
  │    Document results              │
  └───────┬──────────────────────────┘
          │
  ┌───────▼──────────┐
  │ 7. NEXT          │  Select next technique
  │    TECHNIQUE     │  Repeat from step 1
  └──────────────────┘
```

## Key Concepts

| Term | Definition |
|------|------------|
| **Atomic Red Team** | Open-source library by Red Canary containing small, focused tests mapped to MITRE ATT&CK techniques for validating detection capabilities |
| **Invoke-AtomicRedTeam** | PowerShell execution framework for running atomic tests locally or remotely, with prereq checking and cleanup |
| **ATT&CK Navigator** | Web-based tool for annotating and visualizing MITRE ATT&CK matrices as layered heatmaps showing detection coverage |
| **Navigator Layer** | JSON file defining colors, scores, and comments for each ATT&CK technique; used to generate coverage heatmaps |
| **Detection Validation Loop** | Iterative purple team process: execute attack, check for detection, write/tune rule if missed, re-execute to confirm |
| **Coverage Gap** | ATT&CK technique where no detection rule exists or where the rule fails to fire when the technique is executed |
| **Blind Spot** | Technique that was tested via atomic execution but produced zero SIEM alerts, indicating a critical detection failure |
| **Sigma Rule** | Vendor-agnostic detection rule format that can be converted to Splunk SPL, Elastic KQL, Microsoft KQL, and other SIEM queries |
| **Purple Team** | Collaborative security exercise where red team executes known techniques and blue team validates and improves detections in real time |
| **Continuous Atomic Testing** | Scheduled, automated execution of atomic tests to validate that detection rules remain functional as environments change |

## Tools & Systems

- **Invoke-AtomicRedTeam**: PowerShell module for executing Atomic Red Team tests with prereq management and cleanup
- **Atomic Red Team Atomics**: Library of 700+ tests across 200+ MITRE ATT&CK techniques, organized by technique ID
- **ATT&CK Navigator**: Web tool at mitre-attack.github.io/attack-navigator for creating technique coverage heatmaps
- **VECTR**: Purple team reporting platform by SCYTHE for tracking test campaigns, results, and coverage over time
- **Sigma**: Generic signature format for SIEM systems; convert with sigmac or pySigma to target SIEM platforms
- **Splunk / Elastic / Microsoft Sentinel**: SIEM platforms for querying detection results after atomic test execution
- **Sysmon**: Windows system monitoring driver providing detailed process, network, registry, and file telemetry
- **Caldera**: MITRE-developed adversary emulation platform for automated, multi-step attack chains beyond single atomic tests

## Common Scenarios

### Scenario: Validating Credential Access Detections After EDR Deployment

**Context**: The organization deployed a new EDR agent and needs to verify it detects credential dumping techniques (LSASS access, NTDS extraction, SAM dump). The purple team must confirm detections are working before closing the deployment ticket.

**Approach**:
1. Install Invoke-AtomicRedTeam on a test endpoint enrolled in the new EDR
2. Execute credential access atomics: T1003.001 (LSASS Memory), T1003.002 (SAM), T1003.003 (NTDS)
3. Wait 60 seconds, then query EDR console and SIEM for alerts on the test hostname
4. For each technique: mark as DETECTED (green) or BLIND SPOT (red) in the Navigator layer
5. For blind spots, create Sigma rules targeting Sysmon EventCode 10 (ProcessAccess to lsass.exe) and EventCode 1 (known dumping tools)
6. Deploy rules, re-run atomics, confirm detection
7. Export Navigator layer and coverage report for the deployment ticket

**Pitfalls**:
- Running credential dumping atomics without local admin privileges (tests will fail silently)
- Forgetting to enable Sysmon ProcessAccess logging (EventCode 10) which is disabled by default
- Not running cleanup, leaving test artifacts (created accounts, registry keys, scheduled tasks)
- Testing only one sub-technique of T1003 and assuming all credential access is covered
- Not accounting for EDR agent exclusions that may whitelist PowerShell or the AtomicRedTeam directory

### Scenario: Building Monthly ATT&CK Coverage Reporting for Leadership

**Context**: The CISO requires monthly metrics on detection coverage mapped to MITRE ATT&CK. The security team needs to run recurring tests, track improvements, and produce a visual heatmap showing coverage growth over time.

**Approach**:
1. Define a test plan covering the top 50 ATT&CK techniques from Red Canary Threat Detection Report
2. Schedule continuous atomic testing via Windows Task Scheduler running the PowerShell pipeline weekly
3. Export execution logs and SIEM detection query results to JSON after each run
4. Run the Python gap analysis script to produce Navigator layer and coverage report
5. Import the Navigator layer into ATT&CK Navigator, screenshot for executive slide deck
6. Track month-over-month: number of green (detected) vs. red (blind spot) techniques
7. Prioritize detection engineering sprints on the highest-impact blind spots

**Pitfalls**:
- Counting "logged" as "detected" (seeing an event in logs is not the same as having an alert rule)
- Not updating the atomics repository (new tests are added frequently)
- Focusing only on technique count rather than detection quality (a noisy rule with 90% false positives is worse than no rule)
- Not involving the SOC analysts who will triage the alerts in the validation process

## Output Format

```
PURPLE TEAM ATOMIC TESTING REPORT
====================================
Campaign:         Q1 2026 Detection Validation
Date Range:       2026-01-15 to 2026-03-15
Test Environment: YOURLAB-WIN10-01, YOURLAB-SRV-01
SIEM:             Splunk Enterprise 9.x

EXECUTION SUMMARY
Techniques Tested:        47 / 200+ available
Individual Tests Run:     112
Tests Succeeded:          104
Tests Failed (prereqs):   8
Cleanup Completed:        104 / 104

DETECTION COVERAGE
                            Tested    Detected    Coverage
Execution:                     12          10       83.3%
Persistence:                    8           5       62.5%
Credential Access:              6           4       66.7%
Defense Evasion:               10           3       30.0%
Discovery:                      5           5      100.0%
Lateral Movement:               3           1       33.3%
Command and Control:            2           1       50.0%
Exfiltration:                   1           0        0.0%

CRITICAL BLIND SPOTS
[!] T1218.011 - Rundll32          (defense-evasion)
[!] T1218.005 - Mshta             (defense-evasion)
[!] T1070.004 - File Deletion     (defense-evasion)
[!] T1574.001 - DLL Search Order  (persistence)
[!] T1021.002 - SMB/Admin Shares  (lateral-movement)
[!] T1048.003 - Exfil Over HTTP   (exfiltration)

NAVIGATOR LAYER: coverage_layer_20260315.json
  Import at: https://mitre-attack.github.io/attack-navigator/

RECOMMENDATIONS
[CRITICAL] Write Sigma rules for 6 blind spot techniques
[HIGH]     Expand defense-evasion testing (30% coverage)
[HIGH]     Enable exfiltration monitoring data sources
[MEDIUM]   Increase lateral movement test coverage (3 of 7 tested)
```

===== END FILE =====


===== FILE: skills/performing-purple-team-atomic-testing/LICENSE =====

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


===== FILE: skills/performing-purple-team-atomic-testing/references/api-reference.md =====
# API Reference: Purple Team Atomic Testing Agent

## Overview

Parses Atomic Red Team YAML test definitions, correlates execution logs with SIEM detection results, generates MITRE ATT&CK Navigator heatmap layers, computes per-tactic coverage gap analysis, suggests Sigma rules for blind spot techniques, and produces PowerShell retest scripts. Supports the full purple team detection validation loop: execute atomic test, query SIEM for alerts, identify blind spots, write detection rules, and re-validate.

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| pyyaml | >=6.0 | Parsing Atomic Red Team YAML test definitions |

## CLI Usage

```bash
# Full analysis: report + navigator layer + sigma suggestions + retest script
python agent.py --mode all \
  --atomics-path C:\AtomicRedTeam\atomics \
  --log-dir C:\AtomicRedTeam\logs \
  --detections detection_results.json

# Generate only the coverage gap report
python agent.py --mode report \
  --atomics-path C:\AtomicRedTeam\atomics \
  --log-dir C:\AtomicRedTeam\logs \
  --output-report coverage_report.json

# Generate only the ATT&CK Navigator layer
python agent.py --mode navigator \
  --atomics-path C:\AtomicRedTeam\atomics \
  --log-dir C:\AtomicRedTeam\logs \
  --detections detection_results.json \
  --output-layer coverage_layer.json \
  --layer-name "Q1 2026 Purple Team Coverage"

# Generate Sigma rule suggestions for blind spots
python agent.py --mode sigma \
  --atomics-path C:\AtomicRedTeam\atomics \
  --log-dir C:\AtomicRedTeam\logs \
  --detections detection_results.json \
  --output-sigma sigma_suggestions.json

# Generate PowerShell retest script for blind spots
python agent.py --mode retest \
  --atomics-path C:\AtomicRedTeam\atomics \
  --log-dir C:\AtomicRedTeam\logs \
  --detections detection_results.json \
  --output-retest retest_blind_spots.ps1
```

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--atomics-path` | No | Path to Atomic Red Team atomics directory (default: `C:\AtomicRedTeam\atomics`) |
| `--log-dir` | No | Path to atomic test execution log directory (default: `C:\AtomicRedTeam\logs`) |
| `--detections` | No | Path to SIEM detection validation results JSON export |
| `--mode` | No | Output mode: `report`, `navigator`, `sigma`, `retest`, or `all` (default: `all`) |
| `--output-layer` | No | Output path for ATT&CK Navigator layer JSON (default: `navigator_layer.json`) |
| `--output-report` | No | Output path for coverage gap report JSON (default: `coverage_report.json`) |
| `--output-sigma` | No | Output path for Sigma rule suggestions JSON (default: `sigma_suggestions.json`) |
| `--output-retest` | No | Output path for PowerShell retest script (default: `retest_blind_spots.ps1`) |
| `--layer-name` | No | Name for the ATT&CK Navigator layer (default: `Purple Team Coverage`) |

## Key Functions

### `load_atomics_inventory(atomics_path)`
Parses all Atomic Red Team YAML files (`T*/T*.yaml`) from the atomics directory. Returns a dictionary keyed by technique ID containing technique name, test count, supported platforms, executor types, and per-test details (name, description, platforms, executor, elevation required, has cleanup).

### `load_execution_logs(log_dir)`
Loads atomic test execution logs from JSON files matching `T*_*.json` pattern. Tracks per-technique execution count, success/failure counts, hostnames, and last execution timestamp.

### `load_detection_results(detection_file)`
Loads SIEM detection validation results from a JSON file. Each entry maps a technique ID to detection status (detected boolean), alert count, rule name, confidence level (high/medium/low), data sources, SIEM query, and false positive rate.

### `compute_coverage_report(inventory, execution_logs, detection_results)`
Generates a comprehensive coverage gap analysis report including:
- **Executive summary**: total techniques with atomics, executed, detected, and coverage percentages
- **Per-tactic breakdown**: 14 ATT&CK tactics with in-scope techniques, available atomics, executed count, detected count, and detection coverage percentage
- **Gap classification**: blind spots (executed but not detected), not tested (atomics available but not run), and low confidence (detected but unreliable)
- **Prioritized recommendations**: CRITICAL (write rules for blind spots), HIGH (improve low-coverage tactics), MEDIUM (expand testing, tune rules)

### `generate_navigator_layer(inventory, execution_logs, detection_results, layer_name)`
Produces an ATT&CK Navigator v4.5 layer JSON file compatible with `https://mitre-attack.github.io/attack-navigator/`. Color-codes techniques:
- Green (`#66bb6a`): Detected with high/medium confidence
- Yellow (`#ffeb3b`): Detected with low confidence (partial)
- Red (`#ff6666`): Blind spot -- tested but no detection
- Gray (`#d3d3d3`): Not tested -- atomics available

Each technique entry includes metadata (tests available, platforms, executed status, detected status, last execution time, total runs).

### `suggest_sigma_rules(blind_spots)`
Generates Sigma rule stubs for blind spot techniques using built-in templates. Covers common techniques including T1059.001 (PowerShell), T1003.001 (LSASS), T1547.001 (Registry Run Keys), T1053.005 (Scheduled Task), T1070.004 (File Deletion), T1218.011 (Rundll32), and T1105 (Ingress Tool Transfer). Returns rule YAML with title, logsource, detection selection, and ATT&CK tags. Techniques without templates return a manual creation note with ATT&CK reference link.

### `print_coverage_report(report)`
Prints a formatted coverage report to stdout including executive summary, per-tactic coverage table with percentages, critical blind spots list, low-confidence detections, and prioritized recommendations.

### `generate_powershell_test_script(blind_spots, output_path)`
Generates a PowerShell script that re-executes all blind spot techniques using Invoke-AtomicRedTeam with prerequisite checks, execution logging, 30-second SIEM ingestion delays between tests, and cleanup commands. Includes an authorized-testing disclaimer header.

## Detection Results JSON Schema

The `--detections` input file should be a JSON array or object with a `results` key, where each entry contains:

```json
{
  "technique_id": "T1059.001",
  "detected": true,
  "alert_count": 3,
  "rule_name": "Suspicious PowerShell Execution",
  "confidence": "high",
  "data_sources": ["powershell", "sysmon"],
  "siem_query": "index=windows EventCode=4104",
  "false_positive_rate": 0.05
}
```

## ATT&CK Tactic Coverage

The agent tracks 14 ATT&CK Enterprise tactics with top techniques per tactic:

| Tactic | Tactic ID | Techniques Tracked |
|--------|-----------|-------------------|
| Execution | TA0002 | T1059.001, T1059.003, T1047, T1053.005, and 7 more |
| Persistence | TA0003 | T1547.001, T1136.001, T1543.003, T1197, and 8 more |
| Privilege Escalation | TA0004 | T1548.002, T1134.001, T1068, T1055.001, and 2 more |
| Defense Evasion | TA0005 | T1070.001, T1218.011, T1027, T1562.001, and 8 more |
| Credential Access | TA0006 | T1003.001, T1003.002, T1110.001, T1558.003, and 7 more |
| Discovery | TA0007 | T1082, T1083, T1087.001, T1016, and 7 more |
| Lateral Movement | TA0008 | T1021.001, T1021.002, T1021.003, T1570, and 2 more |
| Collection | TA0009 | T1005, T1039, T1074.001, T1113, and 2 more |
| Command and Control | TA0011 | T1071.001, T1105, T1573.001, T1219, and 3 more |
| Exfiltration | TA0010 | T1041, T1048.003, T1567.002 |
| Impact | TA0040 | T1485, T1486, T1489, T1490, T1491.002 |

## Sigma Rule Templates

Built-in templates are provided for high-priority blind spot techniques:

| Technique | Sigma Title | Event Source | Event ID |
|-----------|-------------|-------------|----------|
| T1059.001 | Suspicious PowerShell Script Block Execution | PowerShell | 4104 |
| T1003.001 | LSASS Memory Access for Credential Dumping | Sysmon | 10 |
| T1547.001 | Registry Run Key Persistence | Sysmon | 13 |
| T1053.005 | Scheduled Task Created via Command Line | Security | 4698 |
| T1070.004 | Indicator Removal - File Deletion | Sysmon | 23 |
| T1218.011 | Suspicious Rundll32 Execution | Sysmon | 1 |
| T1105 | Ingress Tool Transfer via Common Utilities | Sysmon | 1 |

===== END FILE =====


===== FILE: skills/performing-purple-team-atomic-testing/scripts/agent.py =====
#!/usr/bin/env python3
"""
Purple Team Atomic Testing Agent

Parses Atomic Red Team YAML definitions, correlates execution logs with detection
results, generates MITRE ATT&CK Navigator layers, and produces coverage gap reports.

Usage:
    python agent.py --atomics-path /path/to/atomics --log-dir /path/to/logs
    python agent.py --atomics-path /path/to/atomics --detections detection_results.json
    python agent.py --mode navigator --output-layer coverage.json
    python agent.py --mode report --output-report coverage_report.json

Requirements:
    pip install pyyaml
"""

import argparse
import hashlib
import json
import math
import os
import re
import sys
from collections import Counter, defaultdict
from datetime import datetime
from pathlib import Path

try:
    import yaml
    HAS_YAML = True
except ImportError:
    HAS_YAML = False


# ---------------------------------------------------------------------------
# MITRE ATT&CK Tactic Metadata
# ---------------------------------------------------------------------------

TACTIC_ORDER = [
    "reconnaissance",
    "resource-development",
    "initial-access",
    "execution",
    "persistence",
    "privilege-escalation",
    "defense-evasion",
    "credential-access",
    "discovery",
    "lateral-movement",
    "collection",
    "command-and-control",
    "exfiltration",
    "impact",
]

TACTIC_ID_MAP = {
    "reconnaissance": "TA0043",
    "resource-development": "TA0042",
    "initial-access": "TA0001",
    "execution": "TA0002",
    "persistence": "TA0003",
    "privilege-escalation": "TA0004",
    "defense-evasion": "TA0005",
    "credential-access": "TA0006",
    "discovery": "TA0007",
    "lateral-movement": "TA0008",
    "collection": "TA0009",
    "command-and-control": "TA0011",
    "exfiltration": "TA0010",
    "impact": "TA0040",
}

# Top ATT&CK techniques by prevalence mapped to their primary tactic
TOP_TECHNIQUES_BY_TACTIC = {
    "execution": [
        "T1059.001", "T1059.003", "T1059.004", "T1059.005",
        "T1059.006", "T1059.007", "T1047", "T1053.005",
        "T1129", "T1203", "T1569.002",
    ],
    "persistence": [
        "T1547.001", "T1547.004", "T1547.009", "T1053.005",
        "T1136.001", "T1543.003", "T1546.001", "T1546.003",
        "T1574.001", "T1574.002", "T1197", "T1505.003",
    ],
    "privilege-escalation": [
        "T1548.002", "T1134.001", "T1068", "T1055.001",
        "T1055.003", "T1055.012",
    ],
    "defense-evasion": [
        "T1070.001", "T1070.004", "T1218.001", "T1218.003",
        "T1218.005", "T1218.010", "T1218.011", "T1027",
        "T1140", "T1562.001", "T1036.005", "T1112",
    ],
    "credential-access": [
        "T1003.001", "T1003.002", "T1003.003", "T1003.004",
        "T1003.005", "T1003.006", "T1110.001", "T1110.003",
        "T1555.003", "T1552.001", "T1558.003",
    ],
    "discovery": [
        "T1082", "T1083", "T1087.001", "T1087.002",
        "T1016", "T1049", "T1057", "T1069.001",
        "T1069.002", "T1518.001", "T1033",
    ],
    "lateral-movement": [
        "T1021.001", "T1021.002", "T1021.003",
        "T1021.004", "T1021.006", "T1570",
    ],
    "collection": [
        "T1005", "T1039", "T1074.001", "T1113",
        "T1115", "T1560.001",
    ],
    "command-and-control": [
        "T1071.001", "T1071.004", "T1105", "T1132.001",
        "T1573.001", "T1219", "T1090.001",
    ],
    "exfiltration": [
        "T1041", "T1048.003", "T1567.002",
    ],
    "impact": [
        "T1485", "T1486", "T1489", "T1490", "T1491.002",
    ],
}


# ---------------------------------------------------------------------------
# Atomics Parsing
# ---------------------------------------------------------------------------

def load_atomics_inventory(atomics_path):
    """Parse all Atomic Red Team YAML files into a technique inventory."""
    if not HAS_YAML:
        print("[ERROR] pyyaml required: pip install pyyaml")
        return {}

    inventory = {}
    atomics_dir = Path(atomics_path)

    if not atomics_dir.exists():
        print(f"[ERROR] Atomics path does not exist: {atomics_path}")
        return {}

    yaml_files = list(atomics_dir.glob("T*/T*.yaml"))
    if not yaml_files:
        print(f"[WARN] No YAML files found in {atomics_path}")
        return {}

    for yaml_file in sorted(yaml_files):
        try:
            with open(yaml_file, "r", encoding="utf-8") as f:
                data = yaml.safe_load(f)

            tech_id = data.get("attack_technique", "")
            if not tech_id:
                continue

            tests = data.get("atomic_tests", [])
            all_platforms = set()
            all_executors = set()
            parsed_tests = []

            for t in tests:
                platforms = t.get("supported_platforms", [])
                executor = t.get("executor", {})
                executor_name = executor.get("name", "unknown")
                all_platforms.update(platforms)
                all_executors.add(executor_name)

                parsed_tests.append({
                    "name": t.get("name", "Unnamed"),
                    "description": t.get("description", ""),
                    "platforms": platforms,
                    "executor": executor_name,
                    "elevation_required": t.get("executor", {}).get(
                        "elevation_required", False
                    ),
                    "has_cleanup": "cleanup_command" in executor,
                })

            inventory[tech_id] = {
                "name": data.get("display_name", tech_id),
                "test_count": len(tests),
                "platforms": sorted(all_platforms),
                "executors": sorted(all_executors),
                "tests": parsed_tests,
                "yaml_path": str(yaml_file),
            }

        except Exception as e:
            print(f"[WARN] Failed to parse {yaml_file.name}: {e}")

    return inventory


def load_execution_logs(log_dir):
    """Load atomic test execution logs from JSON files."""
    executed = {}
    log_path = Path(log_dir)

    if not log_path.exists():
        return executed

    for log_file in sorted(log_path.glob("T*_*.json")):
        try:
            with open(log_file, "r", encoding="utf-8") as f:
                data = json.load(f)

            tech_id = data.get("technique_id", "")
            if not tech_id:
                continue

            if tech_id not in executed:
                executed[tech_id] = {
                    "executions": [],
                    "last_executed": "",
                    "total_runs": 0,
                    "success_count": 0,
                    "failure_count": 0,
                }

            results = data.get("results", [])
            successes = sum(1 for r in results if r.get("status") == "executed")
            failures = sum(1 for r in results if r.get("status") == "failed")

            executed[tech_id]["executions"].append({
                "timestamp": data.get("start_time", ""),
                "hostname": data.get("hostname", "unknown"),
                "username": data.get("username", "unknown"),
                "test_count": len(results),
                "successes": successes,
                "failures": failures,
            })
            executed[tech_id]["total_runs"] += len(results)
            executed[tech_id]["success_count"] += successes
            executed[tech_id]["failure_count"] += failures

            end_time = data.get("end_time", "")
            if end_time > executed[tech_id]["last_executed"]:
                executed[tech_id]["last_executed"] = end_time

        except Exception as e:
            print(f"[WARN] Failed to parse log {log_file.name}: {e}")

    return executed


def load_detection_results(detection_file):
    """Load SIEM detection validation results."""
    if not detection_file or not os.path.exists(detection_file):
        return {}

    try:
        with open(detection_file, "r", encoding="utf-8") as f:
            data = json.load(f)
    except Exception as e:
        print(f"[WARN] Failed to load detections file: {e}")
        return {}

    detections = {}
    entries = data if isinstance(data, list) else data.get("results", [])

    for entry in entries:
        tech_id = entry.get("technique_id", "")
        if not tech_id:
            continue

        detections[tech_id] = {
            "detected": entry.get("detected", False),
            "alert_count": entry.get("alert_count", 0),
            "rule_name": entry.get("rule_name", ""),
            "confidence": entry.get("confidence", "unknown"),
            "data_sources": entry.get("data_sources", []),
            "siem_query": entry.get("siem_query", ""),
            "false_positive_rate": entry.get("false_positive_rate", None),
        }

    return detections


# ---------------------------------------------------------------------------
# Coverage Analysis
# ---------------------------------------------------------------------------

def compute_coverage_report(inventory, execution_logs, detection_results):
    """Generate a comprehensive coverage gap analysis report."""
    report = {
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "summary": {},
        "tactics": {},
        "gaps": {
            "blind_spots": [],       # Executed but not detected
            "not_tested": [],        # Available but not executed
            "low_confidence": [],    # Detected but low confidence
        },
        "recommendations": [],
    }

    total_available = len(inventory)
    total_executed = len(execution_logs)
    total_detected = sum(
        1 for d in detection_results.values() if d.get("detected")
    )
    high_confidence = sum(
        1 for d in detection_results.values()
        if d.get("detected") and d.get("confidence") in ("high", "medium")
    )

    report["summary"] = {
        "total_techniques_with_atomics": total_available,
        "total_techniques_executed": total_executed,
        "total_techniques_with_detection": total_detected,
        "high_confidence_detections": high_confidence,
        "execution_coverage_pct": round(
            (total_executed / total_available * 100) if total_available else 0, 1
        ),
        "detection_coverage_pct": round(
            (total_detected / total_executed * 100) if total_executed else 0, 1
        ),
        "high_confidence_pct": round(
            (high_confidence / total_executed * 100) if total_executed else 0, 1
        ),
    }

    # Per-tactic breakdown
    for tactic in TACTIC_ORDER:
        technique_ids = TOP_TECHNIQUES_BY_TACTIC.get(tactic, [])
        tactic_data = {
            "tactic_id": TACTIC_ID_MAP.get(tactic, ""),
            "techniques_in_scope": len(technique_ids),
            "techniques_with_atomics": 0,
            "techniques_executed": 0,
            "techniques_detected": 0,
            "blind_spots": [],
            "not_tested": [],
        }

        for tech_id in technique_ids:
            has_atomic = tech_id in inventory
            was_executed = tech_id in execution_logs
            detection = detection_results.get(tech_id, {})
            was_detected = detection.get("detected", False)
            confidence = detection.get("confidence", "none")

            if has_atomic:
                tactic_data["techniques_with_atomics"] += 1
            if was_executed:
                tactic_data["techniques_executed"] += 1
            if was_detected:
                tactic_data["techniques_detected"] += 1

            tech_name = inventory.get(tech_id, {}).get("name", tech_id)

            # Classify gaps
            if was_executed and not was_detected:
                gap = {
                    "technique_id": tech_id,
                    "technique_name": tech_name,
                    "tactic": tactic,
                    "status": "BLIND_SPOT",
                }
                tactic_data["blind_spots"].append(tech_id)
                report["gaps"]["blind_spots"].append(gap)

            elif was_detected and confidence == "low":
                gap = {
                    "technique_id": tech_id,
                    "technique_name": tech_name,
                    "tactic": tactic,
                    "status": "LOW_CONFIDENCE",
                    "rule_name": detection.get("rule_name", ""),
                }
                report["gaps"]["low_confidence"].append(gap)

            elif has_atomic and not was_executed:
                gap = {
                    "technique_id": tech_id,
                    "technique_name": tech_name,
                    "tactic": tactic,
                    "status": "NOT_TESTED",
                    "tests_available": inventory[tech_id]["test_count"],
                }
                tactic_data["not_tested"].append(tech_id)
                report["gaps"]["not_tested"].append(gap)

        executed = tactic_data["techniques_executed"]
        detected = tactic_data["techniques_detected"]
        tactic_data["detection_coverage_pct"] = round(
            (detected / executed * 100) if executed else 0, 1
        )
        report["tactics"][tactic] = tactic_data

    # Recommendations
    blind_count = len(report["gaps"]["blind_spots"])
    if blind_count > 0:
        report["recommendations"].append({
            "priority": "CRITICAL",
            "action": f"Create detection rules for {blind_count} blind spot techniques",
            "techniques": [g["technique_id"] for g in report["gaps"]["blind_spots"]],
            "detail": "These techniques were executed but no SIEM/EDR alert was generated",
        })

    low_tactics = [
        t for t, d in report["tactics"].items()
        if d["detection_coverage_pct"] < 30 and d["techniques_executed"] > 0
    ]
    if low_tactics:
        report["recommendations"].append({
            "priority": "HIGH",
            "action": f"Improve detection coverage in: {', '.join(low_tactics)}",
            "detail": "These tactics have less than 30% detection rate among tested techniques",
        })

    untested_count = len(report["gaps"]["not_tested"])
    if untested_count > 10:
        report["recommendations"].append({
            "priority": "MEDIUM",
            "action": f"Expand test execution to {untested_count} untested techniques",
            "detail": "Atomic tests exist but have not been executed yet",
        })

    lc_count = len(report["gaps"]["low_confidence"])
    if lc_count > 0:
        report["recommendations"].append({
            "priority": "MEDIUM",
            "action": f"Tune {lc_count} low-confidence detection rules to reduce false positives",
            "techniques": [g["technique_id"] for g in report["gaps"]["low_confidence"]],
        })

    return report


# ---------------------------------------------------------------------------
# ATT&CK Navigator Layer Generation
# ---------------------------------------------------------------------------

def generate_navigator_layer(inventory, execution_logs, detection_results,
                             layer_name="Purple Team Coverage"):
    """Produce an ATT&CK Navigator v4.5 layer JSON."""
    layer = {
        "name": layer_name,
        "versions": {
            "attack": "15",
            "navigator": "5.1",
            "layer": "4.5",
        },
        "domain": "enterprise-attack",
        "description": (
            f"Purple team atomic testing coverage layer. "
            f"Generated {datetime.utcnow().isoformat()}Z"
        ),
        "filters": {
            "platforms": ["Windows", "Linux", "macOS"],
        },
        "sorting": 0,
        "layout": {
            "layout": "side",
            "aggregateFunction": "average",
            "showID": True,
            "showName": True,
        },
        "hideDisabled": False,
        "techniques": [],
        "gradient": {
            "colors": ["#ff6666", "#ffeb3b", "#66bb6a"],
            "minValue": 0,
            "maxValue": 100,
        },
        "legendItems": [
            {"label": "Blind Spot (tested, no detection)", "color": "#ff6666"},
            {"label": "Partial / Low Confidence", "color": "#ffeb3b"},
            {"label": "Detected (high confidence)", "color": "#66bb6a"},
            {"label": "Not Tested", "color": "#d3d3d3"},
        ],
        "metadata": [],
        "links": [],
        "showTacticRowBackground": True,
        "tacticRowBackground": "#dddddd",
        "selectTechniquesAcrossTactics": True,
        "selectSubtechniquesWithParent": False,
    }

    for tech_id, tech_data in sorted(inventory.items()):
        was_executed = tech_id in execution_logs
        detection = detection_results.get(tech_id, {})
        was_detected = detection.get("detected", False)
        confidence = detection.get("confidence", "none")

        if was_detected and confidence in ("high", "medium"):
            score = 100
            color = "#66bb6a"
            comment = f"DETECTED [{confidence}] - {detection.get('rule_name', 'alert active')}"
        elif was_detected:
            score = 50
            color = "#ffeb3b"
            comment = f"PARTIAL [{confidence}] - detection exists, needs tuning"
        elif was_executed:
            score = 0
            color = "#ff6666"
            comment = "BLIND SPOT - test executed, no detection"
        else:
            score = 0
            color = "#d3d3d3"
            comment = f"NOT TESTED - {tech_data['test_count']} atomic tests available"

        entry = {
            "techniqueID": tech_id,
            "tactic": "",
            "color": color,
            "comment": comment,
            "score": score,
            "enabled": True,
            "metadata": [
                {"name": "tests_available", "value": str(tech_data["test_count"])},
                {"name": "platforms", "value": ", ".join(tech_data["platforms"])},
                {"name": "executed", "value": str(was_executed)},
                {"name": "detected", "value": str(was_detected)},
            ],
            "links": [],
            "showSubtechniques": False,
        }

        if was_executed:
            log = execution_logs[tech_id]
            entry["metadata"].append({
                "name": "last_executed",
                "value": log.get("last_executed", "unknown"),
            })
            entry["metadata"].append({
                "name": "total_runs",
                "value": str(log.get("total_runs", 0)),
            })

        layer["techniques"].append(entry)

    return layer


# ---------------------------------------------------------------------------
# Sigma Rule Suggestion
# ---------------------------------------------------------------------------

SIGMA_TEMPLATES = {
    "T1059.001": {
        "title": "Suspicious PowerShell Script Block Execution",
        "logsource": {"product": "windows", "service": "powershell"},
        "detection_field": "ScriptBlockText",
        "event_id": 4104,
    },
    "T1003.001": {
        "title": "LSASS Memory Access for Credential Dumping",
        "logsource": {"product": "windows", "service": "sysmon"},
        "detection_field": "TargetImage",
        "event_id": 10,
        "target_pattern": "*lsass.exe",
    },
    "T1547.001": {
        "title": "Registry Run Key Persistence",
        "logsource": {"product": "windows", "service": "sysmon"},
        "detection_field": "TargetObject",
        "event_id": 13,
        "target_pattern": "*\\CurrentVersion\\Run*",
    },
    "T1053.005": {
        "title": "Scheduled Task Created via Command Line",
        "logsource": {"product": "windows", "service": "security"},
        "detection_field": "TaskName",
        "event_id": 4698,
    },
    "T1070.004": {
        "title": "Indicator Removal - File Deletion",
        "logsource": {"product": "windows", "service": "sysmon"},
        "detection_field": "TargetFilename",
        "event_id": 23,
    },
    "T1218.011": {
        "title": "Suspicious Rundll32 Execution",
        "logsource": {"product": "windows", "service": "sysmon"},
        "detection_field": "Image",
        "event_id": 1,
        "target_pattern": "*rundll32.exe",
    },
    "T1105": {
        "title": "Ingress Tool Transfer via Common Utilities",
        "logsource": {"product": "windows", "service": "sysmon"},
        "detection_field": "CommandLine",
        "event_id": 1,
        "target_pattern": "*certutil*|*bitsadmin*|*curl*|*wget*",
    },
}


def suggest_sigma_rules(blind_spots):
    """Suggest Sigma rule stubs for blind spot techniques."""
    suggestions = []
    for gap in blind_spots:
        tech_id = gap["technique_id"]
        template = SIGMA_TEMPLATES.get(tech_id)

        if template:
            sigma_stub = {
                "title": template["title"],
                "id": hashlib.md5(tech_id.encode()).hexdigest(),
                "status": "experimental",
                "description": (
                    f"Detects {gap['technique_name']} ({tech_id}) - "
                    f"generated from purple team blind spot analysis"
                ),
                "references": [
                    f"https://attack.mitre.org/techniques/{tech_id.replace('.', '/')}/",
                ],
                "author": "Purple Team Automation",
                "date": datetime.utcnow().strftime("%Y/%m/%d"),
                "tags": [
                    f"attack.{gap.get('tactic', 'unknown')}",
                    f"attack.{tech_id.lower()}",
                ],
                "logsource": template["logsource"],
                "detection": {
                    "selection": {
                        "EventID": template["event_id"],
                    },
                    "condition": "selection",
                },
                "level": "medium",
                "falsepositives": ["Legitimate administrative activity"],
            }

            if "target_pattern" in template:
                sigma_stub["detection"]["selection"][template["detection_field"]] = (
                    template["target_pattern"]
                )

            suggestions.append({
                "technique_id": tech_id,
                "technique_name": gap["technique_name"],
                "sigma_rule": sigma_stub,
            })
        else:
            suggestions.append({
                "technique_id": tech_id,
                "technique_name": gap["technique_name"],
                "sigma_rule": None,
                "note": (
                    f"No template available for {tech_id}. "
                    f"Manual rule creation required. "
                    f"Reference: https://attack.mitre.org/techniques/"
                    f"{tech_id.replace('.', '/')}/"
                ),
            })

    return suggestions


# ---------------------------------------------------------------------------
# Reporting
# ---------------------------------------------------------------------------

def print_coverage_report(report):
    """Print formatted coverage report to stdout."""
    print("=" * 76)
    print("  PURPLE TEAM ATOMIC TESTING - COVERAGE GAP ANALYSIS")
    print("=" * 76)
    print(f"  Generated: {report['generated_at']}")
    print()

    s = report["summary"]
    print("  EXECUTIVE SUMMARY")
    print("  " + "-" * 50)
    print(f"    Techniques with atomics:     {s['total_techniques_with_atomics']}")
    print(f"    Techniques executed:         {s['total_techniques_executed']}")
    print(f"    Techniques with detection:   {s['total_techniques_with_detection']}")
    print(f"    High-confidence detections:  {s['high_confidence_detections']}")
    print(f"    Execution coverage:          {s['execution_coverage_pct']}%")
    print(f"    Detection coverage:          {s['detection_coverage_pct']}%")
    print(f"    High-confidence rate:        {s['high_confidence_pct']}%")
    print()

    print("  PER-TACTIC DETECTION COVERAGE")
    print("  " + "-" * 72)
    header = f"  {'Tactic':<24} {'Scope':>6} {'Avail':>6} {'Exec':>6} {'Det':>6} {'Cov%':>7}"
    print(header)
    print("  " + "-" * 72)

    for tactic in TACTIC_ORDER:
        if tactic not in report["tactics"]:
            continue
        t = report["tactics"][tactic]
        if t["techniques_in_scope"] == 0:
            continue
        cov = t["detection_coverage_pct"]
        indicator = "!!!" if cov < 30 and t["techniques_executed"] > 0 else ""
        print(
            f"  {tactic:<24} {t['techniques_in_scope']:>6} "
            f"{t['techniques_with_atomics']:>6} "
            f"{t['techniques_executed']:>6} "
            f"{t['techniques_detected']:>6} "
            f"{cov:>6.1f}% {indicator}"
        )
    print()

    blind_spots = report["gaps"]["blind_spots"]
    if blind_spots:
        print(f"  CRITICAL BLIND SPOTS ({len(blind_spots)} techniques)")
        print("  " + "-" * 72)
        for gap in blind_spots:
            print(f"    [!] {gap['technique_id']:<14} {gap['technique_name']}")
            print(f"        Tactic: {gap['tactic']}")
        print()

    lc = report["gaps"]["low_confidence"]
    if lc:
        print(f"  LOW-CONFIDENCE DETECTIONS ({len(lc)} techniques)")
        print("  " + "-" * 72)
        for gap in lc:
            rule = gap.get("rule_name", "unnamed rule")
            print(f"    [~] {gap['technique_id']:<14} {gap['technique_name']}")
            print(f"        Rule: {rule} -- needs tuning")
        print()

    if report["recommendations"]:
        print("  RECOMMENDATIONS")
        print("  " + "-" * 72)
        for rec in report["recommendations"]:
            print(f"    [{rec['priority']}] {rec['action']}")
            if "techniques" in rec:
                techs = rec["techniques"][:8]
                suffix = f" (+{len(rec['techniques']) - 8} more)" if len(rec["techniques"]) > 8 else ""
                print(f"           Techniques: {', '.join(techs)}{suffix}")
            if "detail" in rec:
                print(f"           {rec['detail']}")
        print()


def generate_powershell_test_script(blind_spots, output_path):
    """Generate a PowerShell script to re-test blind spot techniques."""
    lines = [
        "# Auto-generated Purple Team Retest Script",
        f"# Generated: {datetime.utcnow().isoformat()}Z",
        f"# Blind spots to retest: {len(blind_spots)}",
        "#",
        "# DISCLAIMER: Only execute on systems you own or have authorization to test.",
        "# These tests execute real attack techniques. Run cleanup after each test.",
        "",
        "Import-Module invoke-atomicredteam",
        "",
        "$Results = @()",
        "",
    ]

    for gap in blind_spots:
        tech_id = gap["technique_id"]
        tech_name = gap["technique_name"]
        lines.extend([
            f'# --- {tech_id}: {tech_name} ---',
            f'Write-Host "[*] Testing {tech_id} - {tech_name}" -ForegroundColor Cyan',
            f'try {{',
            f'    Invoke-AtomicTest {tech_id} -TestNumbers 1 -CheckPrereqs',
            f'    Invoke-AtomicTest {tech_id} -TestNumbers 1 -GetPrereqs',
            f'    Invoke-AtomicTest {tech_id} -TestNumbers 1 -Confirm:$false',
            f'    $Results += [PSCustomObject]@{{ TechniqueId="{tech_id}"; Status="EXECUTED" }}',
            f'    Write-Host "    [+] Success" -ForegroundColor Green',
            f'}} catch {{',
            f'    $Results += [PSCustomObject]@{{ TechniqueId="{tech_id}"; Status="FAILED"; Error=$_.Exception.Message }}',
            f'    Write-Host "    [-] Failed: $($_.Exception.Message)" -ForegroundColor Red',
            f'}}',
            f'Start-Sleep -Seconds 30  # Allow SIEM ingestion',
            f'',
        ])

    lines.extend([
        "# Cleanup all tests",
        'Write-Host "`n[*] Running cleanup..." -ForegroundColor Yellow',
    ])
    for gap in blind_spots:
        tech_id = gap["technique_id"]
        lines.append(
            f'try {{ Invoke-AtomicTest {tech_id} -TestNumbers 1 -Cleanup 2>&1 | Out-Null }} '
            f'catch {{ Write-Host "    Cleanup failed for {tech_id}" -ForegroundColor DarkYellow }}'
        )

    lines.extend([
        "",
        "# Summary",
        '$Results | Format-Table -AutoSize',
        f'$Results | Export-Csv "retest_results_$(Get-Date -Format yyyyMMdd_HHmmss).csv" -NoTypeInformation',
    ])

    with open(output_path, "w", encoding="utf-8") as f:
        f.write("\n".join(lines))

    return output_path


# ---------------------------------------------------------------------------
# Main
# ---------------------------------------------------------------------------

def main():
    parser = argparse.ArgumentParser(
        description="Purple Team Atomic Testing - Coverage Gap Analysis Agent"
    )
    parser.add_argument(
        "--atomics-path",
        default=os.path.join("C:\\", "AtomicRedTeam", "atomics"),
        help="Path to Atomic Red Team atomics directory",
    )
    parser.add_argument(
        "--log-dir",
        default=os.path.join("C:\\", "AtomicRedTeam", "logs"),
        help="Path to atomic test execution logs",
    )
    parser.add_argument(
        "--detections",
        default=None,
        help="Path to SIEM detection validation results JSON",
    )
    parser.add_argument(
        "--mode",
        choices=["report", "navigator", "sigma", "retest", "all"],
        default="all",
        help="Output mode: report, navigator layer, sigma suggestions, retest script, or all",
    )
    parser.add_argument("--output-layer", default="navigator_layer.json",
                        help="Output path for ATT&CK Navigator layer")
    parser.add_argument("--output-report", default="coverage_report.json",
                        help="Output path for coverage report JSON")
    parser.add_argument("--output-sigma", default="sigma_suggestions.json",
                        help="Output path for Sigma rule suggestions")
    parser.add_argument("--output-retest", default="retest_blind_spots.ps1",
                        help="Output path for PowerShell retest script")
    parser.add_argument("--layer-name", default="Purple Team Coverage",
                        help="Name for the ATT&CK Navigator layer")

    args = parser.parse_args()

    print("[*] Purple Team Atomic Testing Agent")
    print(f"    Mode: {args.mode}")
    print()

    # Load data
    print("[*] Loading atomics inventory...")
    inventory = load_atomics_inventory(args.atomics_path)
    print(f"    Loaded {len(inventory)} techniques with atomic tests")

    print("[*] Loading execution logs...")
    exec_logs = load_execution_logs(args.log_dir)
    print(f"    Loaded logs for {len(exec_logs)} techniques")

    print("[*] Loading detection results...")
    det_results = load_detection_results(args.detections)
    print(f"    Loaded detection data for {len(det_results)} techniques")
    print()

    # Generate coverage report
    report = compute_coverage_report(inventory, exec_logs, det_results)

    if args.mode in ("report", "all"):
        print_coverage_report(report)
        with open(args.output_report, "w", encoding="utf-8") as f:
            json.dump(report, f, indent=2)
        print(f"[+] Coverage report saved: {args.output_report}")

    if args.mode in ("navigator", "all"):
        layer = generate_navigator_layer(
            inventory, exec_logs, det_results, args.layer_name
        )
        with open(args.output_layer, "w", encoding="utf-8") as f:
            json.dump(layer, f, indent=2)
        print(f"[+] Navigator layer saved: {args.output_layer}")
        print("    Import at: https://mitre-attack.github.io/attack-navigator/")

    if args.mode in ("sigma", "all"):
        blind_spots = report["gaps"]["blind_spots"]
        if blind_spots:
            suggestions = suggest_sigma_rules(blind_spots)
            with open(args.output_sigma, "w", encoding="utf-8") as f:
                json.dump(suggestions, f, indent=2)
            print(f"[+] Sigma suggestions saved: {args.output_sigma}")
            print(f"    {len([s for s in suggestions if s['sigma_rule']])} rules generated, "
                  f"{len([s for s in suggestions if not s['sigma_rule']])} need manual creation")
        else:
            print("[*] No blind spots found -- no Sigma suggestions needed")

    if args.mode in ("retest", "all"):
        blind_spots = report["gaps"]["blind_spots"]
        if blind_spots:
            ps_path = generate_powershell_test_script(blind_spots, args.output_retest)
            print(f"[+] Retest script saved: {ps_path}")
            print(f"    {len(blind_spots)} techniques queued for retesting")
        else:
            print("[*] No blind spots found -- no retest script needed")

    print()
    print("[*] Done.")


if __name__ == "__main__":
    main()

===== END FILE =====
