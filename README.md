# Detection-Engineering-Lab

A hands-on cybersecurity lab focused on **security telemetry, detection engineering, attack simulation, alert investigation and MITRE ATT&CK mapping** using Windows, Sysmon and Elastic.

---

## Lab Architecture

```text
Windows 10 VM
    │
    ├── Windows Event Logs
    └── Sysmon
            │
            ▼
      Elastic Agent
            │
            ▼
      Elasticsearch
            │
            ▼
         Kibana
            │
            ▼
 Detection & Investigation
```

---

# Lab Setup

## Step 1 — Install Docker

Docker Engine and Docker Compose were installed on the Ubuntu host.

Verify the installation:

```bash
docker --version
docker compose version
```

Verify that Docker is running:

```bash
sudo systemctl status docker
```

---

## Step 2 — Deploy Elasticsearch and Kibana

Elasticsearch and Kibana were deployed on the Ubuntu host using Elastic's local development environment.

```bash
curl -fsSL https://elastic.co/start-local | sh
```

This creates a local Elastic Stack deployment using Docker, including:

* Elasticsearch
* Kibana
* Elasticsearch API endpoint
* Authentication credentials

The deployment was verified by checking the running containers:

```bash
docker ps
```

The main services used by the lab are:

* Elasticsearch: port 9200
* Kibana: port 5601

Kibana is used as the main interface for inspecting and investigating the telemetry collected by the lab.


![Docker running](images/docker_ps.png)

---

## Step 3 — Configure Windows VM

A Windows 10 virtual machine was created using VirtualBox.

### VM Configuration

* **Operating System:** Windows 10
* **RAM:** 3 GB
* **CPU:** 2 vCPUs
* **Virtual Disk:** 60 GB
* **Network:** NAT

Windows 10 is used exclusively as the endpoint generating security telemetry for the lab.

The Windows VM was assigned the following network configuration:

```
Windows VM
IP: 10.0.2.15
Gateway: 10.0.2.2
```

Because the VM uses VirtualBox NAT, 127.0.0.1 from inside Windows refers to the Windows VM itself and not to the Ubuntu host.

Therefore, the Elastic Agent cannot use:

```
127.0.0.1:9200
```

to communicate with Elasticsearch running on Ubuntu.

The Elasticsearch endpoint used by the Agent was:

```
10.0.2.2:9200
```

Connectivity was verified from Windows:

![Testing Connectivity](images/connectivity_test.png)

---

## Step 4 — Configure Windows Security Telemetry

### Windows Event Logs

Windows Event Logs are enabled by default and provide the base telemetry for the lab.

The main log sources used are:

* Security
* System
* Application

---

### Advanced Audit Policy

The following Advanced Audit Policies were configured.

#### Account Logon

**Advanced Audit Policy → Account Logon**

* **Auditar validação de credenciais** → ☑ Success

---

#### Logon/Logoff

**Políticas de Auditoria Avançada → Início de sessão/Fim de sessão**

* **Auditar início de sessão** → ☑ Success + ☑ Failure
* **Auditar encerramento de sessão** → ☑ Success
* **Auditar bloqueio de conta** → ☑ Failure

---

#### Detailed Tracking

**Políticas de Auditoria Avançada → Controlo detalhado**

* **Auditar criação de processos** → ☑ Success

These settings provide additional Windows Security telemetry that will later be collected and analyzed through Elastic.

---

## Step 5 — Install and Configure Sysmon

Microsoft Sysinternals Sysmon was installed to provide detailed endpoint telemetry, particularly process and command-line activity.

### Install Sysmon

After downloading and extracting the Sysmon ZIP:

```cmd
Sysmon64.exe -accepteula -i
```

Sysmon was installed using its default configuration.

### Validate Sysmon Service

```cmd
sc query Sysmon64
```

Expected state:

```text
STATE: 4  RUNNING
```

### Validate Sysmon Driver

```cmd
sc query SysmonDrv
```

Expected state:

```text
STATE: 4  RUNNING
```

### Verify Sysmon Events

Open Event Viewer:

```cmd
eventvwr.msc
```

Navigate to:

**Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**

A test process was executed:

```cmd
notepad.exe
```

The resulting **Sysmon Event ID 1 — Process creation** was verified.

Relevant telemetry fields included:

* `Image`
* `CommandLine`
* `ParentImage`
* `User`
* `Hashes`

At this stage, **Windows Event Logs and Sysmon are generating endpoint telemetry successfully**.

Sysmon Event ID 1 - notepad.exe

![Event Viewer Sysmon Logs](images/event_viewer_id1.png)

---

### Step 6 — Install and Configure Elastic Agent

Elastic Agent was introduced as the collection and forwarding component between the Windows endpoint and Elasticsearch.

The Agent was configured in standalone mode, meaning its configuration is maintained locally rather than being centrally managed by Fleet. Elastic documents standalone agents as locally managed agents using the elastic-agent.yml configuration file.

```text
Windows 10
    │
    ├── Windows Event Logs
    │
    └── Sysmon
          │
          ▼
    Elastic Agent
          │
          ▼
    Elasticsearch
          │
          ▼
       Kibana
```

#### Step 6.1 — Create a Standalone Agent Policy in Kibana

Kibana was used to create an initial standalone Agent policy.

In Kibana:

```
Add Integrations
        ↓
Elastic Agent
        ↓
Create Agent Policy
```

A dedicated policy was created - **Detection-Lab-Windows**

The policy was then used as the starting point for the standalone Agent configuration.

This approach is useful because Kibana prepares the required configuration and integration assets instead of requiring the entire policy to be written manually. Elastic specifically recommends using Kibana to create and download a standalone policy to reduce configuration errors.

Important: the Agent was not enrolled into Fleet. The policy was used as a configuration starting point, while the installed Agent runs in standalone mode.

Agent Policy - Detection-Lab-Windows

![Agent Policies](images/agent_policies.png)


#### Step 6.2 — Select Standalone Mode

In the Kibana Agent setup, Run standalone was selected.

This mode provides the configuration required to install Elastic Agent locally on the Windows machine.

The Agent package and standalone configuration were downloaded/provided through Kiba


### Step 6.3 — Configure Elasticsearch Output


The standalone Agent configuration was stored on Windows at:

```
C:\Program Files\Elastic\Agent\elastic-agent.yml
```

The Elasticsearch output was configured to use the Ubuntu Elasticsearch instance:

```
outputs:
  default:
    type: elasticsearch
    hosts:
      - 10.0.2.2:9200
    api_key: "ID:API_KEY"
    preset: balanced
```

A dedicated API key was created for the Agent with permissions required to monitor Elasticsearch and ingest telemetry.

The API key was then configured in the Agent output.


#### Step 6.4 — Install Elastic Agent on Windows

The Elastic Agent package was extracted on the Windows VM.

From an elevated PowerShell session, the Agent was installed using:

```
.\elastic-agent.exe install
```

The Agent was installed as a Windows service.

The installed configuration is located at:

```
C:\Program Files\Elastic\Agent\elastic-agent.yml
```

#### Step 6.5 — Verify Elastic Agent

The Agent configuration was inspected using:

```
.\elastic-agent.exe inspect
```

This was used to verify:

* Elasticsearch endpoint
* Elasticsearch output
* API key configuration
* Configured inputs

The Agent status was then checked:

![Agent Policies](images/agent_status_healthy.png)


#### Step 6.6 — Configure Windows System Metrics 

The standalone configuration initially included the system/metrics input:

```
- type: system/metrics
  id: unique-system-metrics-input
  data_stream.namespace: default
  use_output: default
  streams:
    - metricsets:
      - cpu
      data_stream.dataset: system.cpu

    - metricsets:
      - memory
      data_stream.dataset: system.memory

    - metricsets:
      - network
      data_stream.dataset: system.network

    - metricsets:
      - filesystem
      data_stream.dataset: system.filesystem
```

This allowed the Agent to collect basic host metrics and also provided an initial way to verify that the Agent could successfully authenticate and send data to Elasticsearch.

The resulting metrics were observed in Kibana, confirming that the Agent-to-Elasticsearch pipeline was functioning.


#### Step 6.7 — Configure Sysmon Collection

After confirming that the Agent was healthy and communicating with Elasticsearch, a winlog input was added to collect the Sysmon Windows Event Log channel.

Sysmon writes its events to:

```
Microsoft-Windows-Sysmon/Operational
```

The Agent configuration was:

```
- type: winlog
  id: sysmon
  use_output: default
  streams:
    - name: Microsoft-Windows-Sysmon/Operational
      data_stream:
        dataset: windows.sysmon_operational
        type: logs
```

The winlog input reads Windows Event Logs through the Windows Event Log API and forwards the events to the configured output.
https://www.elastic.co/docs/reference/fleet/elastic-agent-inputs-list

After applying this configuration I validated the agent status to check if it's still "Healthy"


#### Step 6.8 — Verify Sysmon Ingestion

Once the Sysmon input was enabled, Sysmon events started appearing in Elasticsearch/Kibana.

The events were observed under the dataset:

```
windows.sysmon_operational
```

Kibana Discover showing windows.sysmon_operational events:

![Sysmon Events](images/sysmon_operational.png)




### Step 7 — Log Ingestion & Normalization

* Verify events arriving in Elasticsearch
* Explore events in Kibana Discover
* Validate ECS fields
* Investigate fields such as:

  * `host.name`
  * `agent.name`
  * `event.code`
  * `process.name`
  * `process.command_line`
  * `source.ip`
  * `destination.ip`

### Step 8 — Detection Engineering

* Create detection rules
* Implement Sigma rules
* Map detections to MITRE ATT&CK
* Tune detections and reduce false positives

### Step 9 — Attack Simulation

Controlled simulations will be performed to generate telemetry for:

* Failed authentication
* Suspicious PowerShell
* Process execution
* Persistence
* Network activity
* Other controlled adversary behaviors

### Step 10 — Detection Validation

For each simulated behavior:

```text
Attack
  ↓
Telemetry
  ↓
Detection
  ↓
Alert
  ↓
Investigation
  ↓
Response
```

### Step 11 — Investigation & Response

* Investigate alerts in Kibana
* Correlate related events
* Identify MITRE ATT&CK techniques
* Document findings
* Define appropriate response actions

### Step 12 — Documentation

The final lab documentation will include:

* Architecture
* Attack simulations
* Telemetry generated
* Detection rules
* MITRE ATT&CK mappings
* Investigation process
* Response actions
* Screenshots
* Results and limitations

