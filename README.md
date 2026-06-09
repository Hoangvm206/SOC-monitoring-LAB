# SOC Monitoring & Incident Response Lab

## Overview

This project demonstrates the deployment of a Security Operations Center (SOC) laboratory environment using open-source technologies. The objective is to centralize log collection, detect suspicious activities across Windows and Linux endpoints, and manage security incidents through a SIEM and SOAR workflow.

Key capabilities include:

* Centralized log collection and monitoring
* Windows telemetry using Sysmon
* Linux telemetry using Auditd
* Custom detection engineering
* MITRE ATT&CK mapping
* Attack simulation and validation
* Incident management using TheHive 5

---

## Architecture

```text
Windows Endpoint (Sysmon)
            │
            ▼
       Wazuh Agent
            │
            ▼
      Wazuh Manager
            │
            ▼
     Wazuh Dashboard
            │
            ▼
         TheHive 5
            │
            ▼
       SOC Analyst
```

> 📸 Add architecture diagram here

---

## Technologies Used

| Component          | Technology                            |
| ------------------ | ------------------------------------- |
| SIEM               | Wazuh 4.9.x                           |
| SOAR               | TheHive 5                             |
| Windows Monitoring | Sysmon                                |
| Linux Monitoring   | Auditd                                |
| Visualization      | OpenSearch Dashboard                  |
| Containerization   | Docker                                |
| Virtualization     | VirtualBox                            |
| Operating Systems  | Windows 10, Ubuntu Server, Kali Linux |

---

## Project Structure

### Phase 1 – Wazuh Deployment

* Deploy Wazuh SIEM using Docker
* Configure centralized log collection
* Connect Windows endpoint
* Troubleshoot networking issues

---

### Phase 2 – Windows Monitoring

* Deploy Sysmon
* Configure EventChannel collection
* Develop custom detection rules
* Map detections to MITRE ATT&CK

Sample detections:

* Account Discovery (whoami)
* PowerShell Hidden Execution

---

### Phase 3 – Cross-Platform Monitoring & Attack Simulation

* Deploy Linux Wazuh Agent
* Configure Auditd
* Collect Windows and Linux telemetry
* Simulate adversary techniques
* Validate detection coverage

Covered ATT&CK techniques:

* T1033 – Account Discovery
* T1053 – Scheduled Task / Cron
* T1136 – Create Account
* T1070 – Clear History / Defense Evasion
* T1218 – LOLBins Execution

---

### Phase 4 – TheHive Integration & Incident Handling

* Integrate Wazuh with TheHive 5
* Forward alerts automatically
* Create investigation cases
* Perform alert triage
* Manage incident lifecycle

Incident workflow:

```text
Alert
 ↓
Assign
 ↓
Triage
 ↓
Case Creation
 ↓
Investigation
 ↓
Closure
```

---

## Detection Coverage

| Platform | Data Source         |
| -------- | ------------------- |
| Windows  | Sysmon              |
| Windows  | Event Logs          |
| Linux    | Auditd              |
| Linux    | Authentication Logs |

ATT&CK Coverage:

* Discovery
* Persistence
* Execution
* Defense Evasion
* Account Manipulation

---

## Key Achievements

* Centralized monitoring across Windows and Linux endpoints.
* Developed custom Wazuh detection rules.
* Implemented MITRE ATT&CK-aligned alerting.
* Simulated real-world attack techniques.
* Integrated SIEM and SOAR workflows.
* Established an end-to-end incident response process.

---

## Future Improvements

* Threat Intelligence Integration
* VirusTotal Enrichment
* Active Response
* Automated Playbooks
* Autonomous SOC Framework

---

## Author

Vu Hoang

Cybersecurity Student | 
