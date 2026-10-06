# Windows — PowerShell Execution Policy Bypass

## 1. Objective

The objective of this detection is to identify PowerShell processes executed with the `ExecutionPolicy` parameter set to `Bypass`.

PowerShell execution with an explicit execution policy bypass can be associated with attempts to circumvent PowerShell execution restrictions and may be observed during malicious script execution.

The detection is designed to identify this specific command-line behavior and provide an alert for further investigation.

---

## 2. Rule Configuration and Detection Logic

The detection uses **Sysmon Event ID 1 — Process Creation**, collected from the Windows laboratory machine by Elastic Agent.

The detection was implemented in Kibana using an **Elasticsearch query** rule.

### KQL

```kql
event.code: "1" AND
winlog.event_data.Image: *powershell.exe AND
winlog.event_data.CommandLine: *-ExecutionPolicy* AND
winlog.event_data.CommandLine: *Bypass*
```

### Rule Configuration

| Setting        | Value                                                                                                                                                            |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Data view      | `logs-*`                                                                                                                                                         |
| Query          | `event.code: "1" AND winlog.event_data.Image: *powershell.exe AND winlog.event_data.CommandLine: *-ExecutionPolicy* AND winlog.event_data.CommandLine: *Bypass*` |
| Condition      | Alert when matches are found                                                                                                                                     |
| Check interval | 1 minute                                                                                                                                                         |
| Rule name      | `Windows - PowerShell Execution Policy Bypass`                                                                                                                   |

The rule generates an alert whenever a matching Sysmon Event ID 1 is observed.

Unlike the **Multiple Failed Logons** detection, this detection does not require multiple events. A single PowerShell process executed with `-ExecutionPolicy Bypass` is sufficient to trigger the alert.

---

## 3. MITRE ATT&CK Mapping

### T1059.001 — PowerShell

The detection is mapped to **MITRE ATT&CK T1059.001 — PowerShell**.

PowerShell is a command and scripting interpreter commonly used by both administrators and adversaries. Its use with an explicit `ExecutionPolicy Bypass` parameter can be relevant when investigating potentially malicious script execution or attempts to circumvent execution restrictions.

The MITRE ATT&CK mapping represents the **technique associated with the observed behavior**. It does not by itself confirm malicious activity.

---

## 4. Attack Simulation and Detection Validation

A controlled PowerShell command was executed on the Windows laboratory machine using an explicit execution policy bypass:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Get-Process"
```

The event matched the detection query and generated an alert in Kibana.

![Powershell alert](../../images/alert_powershell.png)

The corresponding event was then located in the ingested Sysmon telemetry, confirming the complete detection pipeline:

```text
PowerShell Execution
        ↓
Sysmon Event ID 1
        ↓
Elastic Agent
        ↓
Elasticsearch
        ↓
KQL Detection
        ↓
Kibana Alert
        ↓
Matching Event
```

This validated that the simulated behavior generated the expected telemetry and successfully triggered the detection.

---

## 5. Investigation

When an alert is generated, the analyst should investigate the process creation event and its surrounding context.

Relevant fields include:

| Field                           | Purpose                                         |
| ------------------------------- | ----------------------------------------------- |
| `event.code`                    | Confirms the Sysmon event type                  |
| `host.name`                     | Identifies the affected endpoint                |
| `winlog.event_data.Image`       | Identifies the executed process                 |
| `winlog.event_data.CommandLine` | Shows the PowerShell command and parameters     |
| `winlog.event_data.ParentImage` | Identifies the process that launched PowerShell |
| `winlog.event_data.User`        | Identifies the user associated with the process |
| `@timestamp`                    | Establishes when the process was executed       |

The analyst should review the complete command line to determine what PowerShell was executing and why the execution policy was bypassed.

The parent process should also be reviewed because the process that launched PowerShell can provide important context. For example, PowerShell launched interactively by an administrator may have a different risk profile from PowerShell spawned by an unexpected application, document, script, or other process.

Additional investigation should consider:

* Whether the execution was expected
* Which user executed PowerShell
* The parent process and process ancestry
* The complete PowerShell command line
* Other PowerShell activity occurring around the same time
* Related process creation events
* Network connections or other activity associated with the execution
* Whether similar activity occurred on other hosts

The presence of `-ExecutionPolicy Bypass` alone should not be considered proof of malicious activity.

The following screenshot shows the Windows Security events ingested into Elastic and the fields used during the investigation.

![Elastic Logs](../../images/Elastic_logs_powershell.png)

---

## 6. Response Considerations

Potential response actions depend on the investigation results and organizational procedures.

If the activity is determined to be legitimate, the event can be documented as expected administrative or operational activity and considered for tuning if appropriate.

If the activity is considered suspicious, potential actions may include:

* Investigating the originating user and host
* Reviewing the PowerShell command and parent process
* Investigating related process execution
* Checking for persistence or additional suspicious activity
* Reviewing network activity associated with the process
* Escalating the activity for further investigation
* Containing the affected endpoint if malicious activity is confirmed

---

## 7. Limitations

The detection has several limitations:

* `-ExecutionPolicy Bypass` can be used legitimately by administrators and applications.
* The detection does not determine whether the executed PowerShell code is malicious.
* The detection does not inspect the contents of scripts executed by PowerShell.
* The detection does not currently correlate the PowerShell process with subsequent activity.
* The current rule was validated in a controlled laboratory environment and has not been tuned for a production environment.
* The detection may generate false positives in environments where PowerShell execution with an explicit execution policy bypass is legitimate.

Future improvements could include correlation with additional PowerShell telemetry, such as PowerShell Script Block Logging, and additional process or network activity.

---

## 8. Status

**Status:** Validated

The detection was successfully tested end-to-end:

```text
PowerShell Execution
        ↓
Sysmon Event ID 1
        ↓
Elastic Agent
        ↓
Elasticsearch
        ↓
KQL Detection
        ↓
Kibana Alert
        ↓
Matching Event
```
