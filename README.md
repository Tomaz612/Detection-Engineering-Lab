# Detection Engineering Lab

A hands-on cybersecurity lab focused on **detection engineering, security telemetry, attack simulation and alert investigation** using Elastic, Windows and Sysmon.

The goal is to build **practical, testable and explainable detections** and validate them through controlled attack simulations.

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

---

## Detections

| ID      | Detection                                                                                           | Data Source           | MITRE ATT&CK           | Status      |
| ------- | --------------------------------------------------------------------------------------------------- | --------------------- | ---------------------- | ----------- |
| DET-001 | [Multiple Failed Logons](detections/windows-multiple-failed-logons.md)                              | Windows Security 4625 | T1110 — Brute Force    | ✅ Validated |
| DET-002 | [PowerShell Execution Policy Bypass](detections/windows-powershell-execution-policy-bypass.md)      | Sysmon 1              | T1059.001 — PowerShell | ✅ Validated |

More detections will be added as the laboratory evolves.

---

## Detection Workflow

Each detection follows the same development lifecycle:

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

The focus is not only on generating alerts, but on understanding the telemetry behind them and determining whether detected activity is expected, suspicious or potentially malicious.

---

## MITRE ATT&CK Coverage

![MITRE ATT\&CK Coverage](images/mitre-coverage.png)

The detections are mapped to the relevant MITRE ATT&CK techniques to provide context around the adversary behaviors being monitored.

---

## Lab Environment

* **Ubuntu 24.04** — Elastic host
* **Docker** — Elastic deployment
* **Elasticsearch 9.5.4** — telemetry storage and search
* **Kibana 9.5.4** — detection, alerting and investigation
* **Windows 10 VM** — monitored endpoint
* **Sysmon** — endpoint telemetry
* **Elastic Agent** — telemetry collection

[View Lab Setup →](docs/lab-setup.md)

---

## Detection Playbooks

Each detection has a dedicated playbook containing:

* Detection objective
* Detection logic
* MITRE ATT&CK mapping
* Validation
* Investigation
* Limitations
* Evidence

[View Detection Playbooks →](detections/)

---

## Project Goals

This project focuses on:

* Building detections from observable security telemetry
* Developing behavioral rather than purely signature-based detections
* Validating detections through controlled simulations
* Investigating the underlying telemetry
* Identifying false positives
* Tuning detection logic
* Mapping detections to MITRE ATT&CK
* Documenting the detection lifecycle

The long-term goal is to build a growing collection of **tested and documented Elastic detections** covering different adversary behaviors.

---

## Status

**Active — Detection Engineering Lab**

New detections, simulations and tuning exercises will be added over time.

