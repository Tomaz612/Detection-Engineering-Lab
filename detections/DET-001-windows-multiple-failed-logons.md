# DET-001 — Windows Multiple Failed Logons

## Objective

Detect multiple failed Windows authentication attempts that may indicate brute-force or password-guessing activity.

The detection uses Windows Security **Event ID 4625**.

## Detection Logic

The rule uses a threshold-based detection:

| Setting            | Value                         |
| ------------------ | ----------------------------- |
| Event              | `4625`                        |
| Group by           | `winlog.event_data.IpAddress` |
| Threshold          | `>= 5`                        |
| Time window        | 5 minutes                     |
| Execution interval | 5 minutes                     |

An alert is generated when **5 or more Event ID 4625 events are observed from the same source IP within 5 minutes**.

## MITRE ATT&CK

* **Tactic:** Credential Access
* **Technique:** T1110 — Brute Force
* **Procedure:** Detect repeated failed authentication attempts from the same source IP, which may indicate password guessing or brute-force activity.

The detection identifies behavior associated with brute-force activity but does not by itself confirm malicious intent. Legitimate authentication failures may also trigger the rule.

## Validation

The detection was validated by generating multiple failed authentication attempts on the Windows lab endpoint.

The events were successfully:

1. Generated as Windows Event ID 4625.
2. Collected by Elastic Agent.
3. Ingested into Elasticsearch.
4. Matched by the detection rule.
5. Correlated by the threshold condition.
6. Converted into a Kibana alert.

![Multiple Failed Logons Alert](../../images/alert_Windows_Multiple_Failed.png)

## Investigation

Relevant fields for investigation include:

* `event.code`
* `host.name`
* `winlog.event_data.IpAddress`
* `winlog.event_data.TargetUserName`
* `winlog.event_data.LogonType`
* `winlog.event_data.WorkstationName`

The alert should be investigated in context before determining whether the activity is malicious.

![Elastic Logs](../../images/Elastic_Logs_4625.png)

## Limitations

* Legitimate authentication failures may trigger the detection.
* Failed and successful authentication events are not currently correlated.
* The threshold has only been validated in a controlled lab environment.
* Threshold values may require tuning for production environments.

## Status

**Validated**

