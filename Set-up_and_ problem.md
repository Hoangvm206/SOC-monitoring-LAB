# SOC Lab Deployment: Wazuh SIEM Integration & VirtualBox Network Troubleshooting

## Overview

This project documents the deployment of a small Security Operations Center (SOC) Lab designed for centralized log collection, analysis, and endpoint monitoring using the open-source Wazuh SIEM platform.

### Architecture

```text
+------------------------+
|   Wazuh SIEM Server    |
| Ubuntu Server 22.04    |
| Wazuh 4.9.2            |
+-----------+------------+
            |
            |
    Host-Only Network
   (192.168.56.0/24)
            |
            |
+-----------+------------+
|     Windows 10 VM      |
|     Wazuh Agent        |
|       4.9.2            |
+------------------------+
```

### Components

| Component       | Description                               |
| --------------- | ----------------------------------------- |
| SIEM Server     | Ubuntu Server 22.04 running Wazuh 4.9.2   |
| Wazuh Manager   | Receives logs and manages agents          |
| Wazuh Indexer   | Stores and indexes collected logs         |
| Wazuh Dashboard | Web interface for monitoring and analysis |
| Endpoint        | Windows 10 Enterprise LTSC                |
| Wazuh Agent     | Collects and forwards logs to the Manager |

---

# Troubleshooting Log

The deployment process involved several networking and system compatibility issues. The following section summarizes the root causes and resolutions.

## Issue 1: Missing `ossec.conf` After Silent Installation

### Symptoms

* Wazuh Agent installation completed successfully.
* Windows service `WazuhSvc` was running.
* Dashboard displayed:

```text
No agents were added
```

* Configuration file `ossec.conf` was missing.

### Root Cause

Silent installation using the `/q` parameter caused Windows Installer to complete without properly generating the required configuration files. In some cases, files may be redirected to VirtualStore or remain incomplete due to previous installations.

### Resolution

1. Completely uninstall the agent.
2. Remove any remaining agent files.
3. Reinstall using the graphical installation wizard.
4. Verify that:

```text
C:\Program Files (x86)\ossec-agent\
```

contains the generated configuration files.

---

## Issue 2: Enrollment Failure Due to Localhost Configuration

### Symptoms

Agent logs reported:

```text
Unable to connect to enrollment service at [127.0.0.1]:1515
```

### Root Cause

The VirtualBox NAT Port Forwarding configuration used:

```text
Host IP = 127.0.0.1
```

This restricted listening to the host machine's loopback interface only. The Windows VM could not reach the Ubuntu server through this address.

### Resolution

1. Remove the Host IP value from Port Forwarding rules.
2. Determine the NAT Gateway address:

```text
10.0.2.2
```

3. Configure the agent to connect to:

```xml
<address>10.0.2.2</address>
```

4. Re-enroll the agent.

---

## Issue 3: Agent Appears Registered but Never Connects

### Symptoms

* Agent successfully enrolled.
* Agent visible in Dashboard.
* Status remained:

```text
Never connected
```

or

```text
Disconnected
```

### Root Cause

The default Docker configuration exposed:

```yaml
1514-1515:1514-1515/tcp
```

However, Wazuh Agents may send keep-alive messages through UDP 1514 depending on configuration. Since UDP 1514 was not exposed, packets were dropped before reaching the Manager.

### Resolution

Modify the Wazuh Manager service in `docker-compose.yml`:

```yaml
ports:
  - "1514:1514/tcp"
  - "1514:1514/udp"
  - "1515:1515/tcp"
```

Restart the stack:

```bash
docker compose down
docker compose up -d
```

---

## Issue 4: Migrating to Host-Only Networking

### Symptoms

Although enrollment succeeded, communication over VirtualBox NAT remained unstable and agents frequently disconnected.

### Root Cause

NAT and Port Forwarding introduced additional complexity and made troubleshooting more difficult during lab operation.

### Resolution

#### Network Migration

Move both virtual machines to:

```text
Host-Only Adapter
```

Configuration:

```text
Network: 192.168.56.0/24
Promiscuous Mode: Allow All
```

#### Protocol Standardization

Update the agent configuration:

```xml
<protocol>tcp</protocol>
```

Using TCP provides reliable delivery and simplifies troubleshooting.

---

# Final Result

After reconfiguring the environment:

* Wazuh Agent connected successfully.
* Communication established through:

```text
192.168.56.101:1514/tcp
```

* Agent status changed to:

```text
Active
```

* Windows logs were successfully collected:

  * Security Logs
  * System Logs
  * Application Logs

The environment is now ready for additional telemetry sources such as Sysmon.
![alt text](<Screenshot 2026-06-01 203625.png>)

---

# Lessons Learned

## 1. Networking Is the Foundation of Security Monitoring

Understanding how packets move between systems is essential for building and maintaining monitoring infrastructure.

This lab provided practical experience with:

* NAT
* Port Forwarding
* Loopback Addresses (127.0.0.1)
* Host-Only Networks
* TCP vs UDP Communication

---

## 2. Log Analysis Is Critical

Every troubleshooting step was guided by log analysis rather than trial-and-error.

Key files included:

```text
ossec.log
```

Carefully reviewing connection errors, enrollment messages, and protocol mismatches significantly reduced troubleshooting time.

---

## 3. Never Assume Default Configurations Are Correct

Default configurations are designed for common deployment scenarios and may not fit a virtualized lab environment.

Successful deployment required:

* Adjusting Docker port mappings
* Modifying network topology
* Switching communication protocols
* Verifying service compatibility

Understanding and adapting default settings is an important skill for security engineers and SOC analysts.

---

# Technologies Used

* Ubuntu Server 22.04 LTS
* Windows 10 Enterprise LTSC
* Wazuh 4.9.2
* Docker Engine
* Docker Compose V2
* VirtualBox
* Host-Only Networking
* TCP/IP
* Sysmon (planned integration)
