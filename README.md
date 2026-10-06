# Detection-Engineering-Lab

A hands-on cybersecurity lab focused on **detection engineering, security telemetry, attack simulation, alert investigation and MITRE ATT&CK mapping** using **Elastic, Windows and Sysmon**.

The main objective is to build and validate detections from end to end:

```text
Adversary Behavior
        ↓
Telemetry
        ↓
Detection
        ↓
Alert
        ↓
Investigation
        ↓
Tuning
        ↓
Documentation
```

The project focuses on building **practical, testable and explainable security detections**, rather than simply collecting logs.

---

## Architecture

```text
┌──────────────────────┐
│     Windows VM       │
│                      │
│ Windows Event Logs   │
│ Sysmon               │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    Elastic Agent     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│    Elasticsearch     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│       Kibana         │
│                      │
│ Detection & Alerting │
│ Investigation        │
└──────────────────────┘
```

The laboratory currently collects Windows Security and Sysmon telemetry into Elasticsearch, where detections are developed and investigated through Kibana.

---

# Detection Engineering

The core of this project is the development and validation of Elastic detections.

Each detection follows a consistent workflow:

```text
Define Behavior
      ↓
Identify Telemetry
      ↓
Implement Detection
      ↓
Simulate Behavior
      ↓
Validate Detection
      ↓
Investigate Alert
      ↓
Tune Detection
      ↓
Document Results
```

## Implemented Detections

| Detection                                                                                           | Data Source      | Event ID | MITRE ATT&CK           | Status      |
| --------------------------------------------------------------------------------------------------- | ---------------- | -------: | ---------------------- | ----------- |
| [Multiple Failed Logons](docs/detections/windows-multiple-failed-logons.md)                         | Windows Security |     4625 | T1110 — Brute Force    | ✅ Validated |
| [PowerShell Execution Policy Bypass](docs/detections/windows-powershell-execution-policy-bypass.md) | Sysmon           |        1 | T1059.001 — PowerShell | ✅ Validated |

More detections will be added as the laboratory evolves.

---

## Detection Examples

### Windows — Multiple Failed Logons

Detects repeated Windows authentication failures that may indicate password guessing or brute-force activity.

**Telemetry**

```text
Windows Security Event ID 4625
```

**Detection**

```text
5+ failed logons
within 5 minutes
grouped by targeted account
```

**MITRE ATT&CK**

```text
T1110 — Brute Force
```

The detection was validated by generating controlled failed authentication attempts and confirming that the expected telemetry triggered the Elastic alert.

[View Detection Playbook →](docs/detections/windows-multiple-failed-logons.md)

---

### Windows — PowerShell Execution Policy Bypass

Detects PowerShell execution using an explicit `-ExecutionPolicy Bypass` parameter.

**Telemetry**

```text
Sysmon Event ID 1 — Process Creation
```

**Detection**

```kql
event.code: "1" AND
winlog.event_data.Image: *powershell.exe AND
winlog.event_data.CommandLine: *-ExecutionPolicy* AND
winlog.event_data.CommandLine: *Bypass*
```

**MITRE ATT&CK**

```text
T1059.001 — PowerShell
```

The detection was validated by executing PowerShell with an explicit execution policy bypass and confirming that the resulting Sysmon event triggered the Elastic alert.

[View Detection Playbook →](docs/detections/windows-powershell-execution-policy-bypass.md)

---

# Attack Simulation

Detections are validated through controlled simulations performed inside the isolated Windows laboratory environment.

Current simulation categories include:

* Failed authentication
* PowerShell execution
* Process execution
* Persistence
* Network activity
* Other controlled adversary behaviors

Each simulation is linked to a detection and its corresponding playbook.

The objective is not simply to generate an alert, but to verify the complete chain:

```text
Simulated Behavior
        ↓
Expected Telemetry
        ↓
Elastic Detection
        ↓
Alert
        ↓
Investigation
```

---

# Investigation & Validation

Detection validation goes beyond confirming that an alert was generated.

For each detection, the investigation process considers:

* Affected host
* User or account
* Process and command line
* Parent process
* Source information
* Related events
* Potential false positives
* Additional activity surrounding the alert

The goal is to determine whether the detected behavior is **expected, suspicious or potentially malicious**.

Screenshots and detailed investigation evidence are maintained within the individual detection playbooks.

---

# MITRE ATT&CK

Detections are mapped to **MITRE ATT&CK techniques** based on the behavior they are designed to identify.

Current mappings:

| Detection                          | Technique              |
| ---------------------------------- | ---------------------- |
| Multiple Failed Logons             | T1110 — Brute Force    |
| PowerShell Execution Policy Bypass | T1059.001 — PowerShell |

The mappings describe the adversary behavior represented by the detection and do not imply that every alert corresponds to confirmed malicious activity.

---

# Lab Environment

The laboratory uses:

* **Ubuntu** — Elastic host
* **Docker** — Elastic deployment
* **Elasticsearch** — telemetry storage and search
* **Kibana** — detection, alerting and investigation
* **Windows 10 VM** — monitored endpoint
* **Sysmon** — endpoint process and command-line telemetry
* **Elastic Agent** — telemetry collection

Detailed setup and configuration documentation is maintained separately to keep this README focused on the detection engineering work.

[View Lab Setup →](docs/lab-setup.md)

---

# Detection Playbooks

Each completed detection has a dedicated playbook containing the technical implementation and validation details.

Playbooks document:

* Detection objective
* Telemetry source
* Detection logic
* MITRE ATT&CK mapping
* Attack simulation
* Expected telemetry
* Detection validation
* Investigation procedure
* False-positive considerations
* Tuning opportunities
* Response considerations
* Evidence

### Available Playbooks

* [Windows — Multiple Failed Logons](docs/detections/windows-multiple-failed-logons.md)
* [Windows — PowerShell Execution Policy Bypass](docs/detections/windows-powershell-execution-policy-bypass.md)

---

# Project Goals

The project is being developed around several practical detection engineering principles:

* Build detections from observable security telemetry
* Prefer specific behavioral detections over generic alerts
* Validate detections through controlled simulations
* Investigate the underlying telemetry
* Identify and document false positives
* Tune detections based on observed behavior
* Map detections to MITRE ATT&CK
* Document the complete detection lifecycle

The long-term goal is to build a growing collection of **tested and documented Elastic detections** covering different adversary behaviors.

---

## Status

**Active — Detection Engineering Lab**

Current focus:

```text
Telemetry
   ↓
Elastic Detection Rules
   ↓
Alert Validation
   ↓
Investigation
   ↓
Tuning
   ↓
MITRE ATT&CK
   ↓
Documentation
```

