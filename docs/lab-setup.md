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


## Step 6 — Configure Elastic Agent

Elastic Agent was installed on the Windows endpoint in **standalone mode** and configured to collect Windows Security and Sysmon telemetry and forward it to Elasticsearch.

### Agent Architecture

```text
Windows Endpoint
      │
      ├── Windows Security
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

### Installation

Elastic Agent was installed as a Windows service on the endpoint.

The standalone configuration file is located at:

```text
C:\Program Files\Elastic\Agent\elastic-agent.yml
```

The Elasticsearch output was configured to use the lab's Elasticsearch instance.

### Telemetry Inputs

The agent was configured with two `winlog` inputs:

**Windows Security**

```yaml
- type: winlog
  id: security
  use_output: default
  streams:
    - name: Security
      data_stream:
        dataset: windows.security
        type: logs
```

**Sysmon**

```yaml
- type: winlog
  id: sysmon
  use_output: default
  streams:
    - name: Microsoft-Windows-Sysmon/Operational
      data_stream:
        dataset: windows.sysmon_operational
        type: logs
```

These inputs allow the lab to collect:

* Windows Security events
* Sysmon process creation and other endpoint telemetry

### Verification

After restarting the Elastic Agent service, the agent status was verified in Kibana and reported as **HEALTHY**.

The collected telemetry was then verified in Kibana using the following datasets:

```text
windows.security
windows.sysmon_operational
```

At this point, the Windows endpoint was successfully connected to Elasticsearch and ready for detection engineering.

### Result

The lab now has the following telemetry pipeline:

```text
Windows Security ──┐
                   ├──> Elastic Agent ──> Elasticsearch ──> Kibana
Sysmon ────────────┘
```

This telemetry will be used as the data source for the detection rules implemented in the lab.



