# Splunk Forwarder Setup and Log Forwarding

## Overview

This lab demonstrates the configuration of centralized log collection using the Splunk Universal Forwarder. The forwarder was installed on a client machine (CLIENT01) and configured to send Windows Event Logs to a centralized Splunk instance on SRV01.

This lab also included troubleshooting steps to resolve configuration issues preventing log ingestion.

## Objective

- Install Splunk Universal Forwarder on CLIENT01
- Configure forwarding to SRV01
- Enable monitoring of Windows Event Logs
- Verify centralized log ingestion in Splunk
- Troubleshoot configuration issues preventing log forwarding

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01 (Splunk Enterprise)
- Client Machine: CLIENT01 (Splunk Universal Forwarder)
- Network: LAB-NET (Internal) + NAT (Internet access)

## Steps Performed

### Step 1: Prepare Network Connectivity

CLIENT01 did not initially have internet access required to download the Splunk Universal Forwarder.

To resolve this:

1. Powered off CLIENT01
2. Opened **VirtualBox** → **Settings** → **Network**
3. Verified:
   - Adapter 1 → Internal Network (`NET-LAB`)
4. Enabled Adapter 2:
   - Attached to: **NAT**
5. Started CLIENT01
6. Verified internet connectivity

```cmd
ipconfig
ping 8.8.8.8
```

> 💡 Adapter 1 maintains domain communication, while Adapter 2 provides internet access.

### Step 2: Install Splunk Universal Forwarder

On CLIENT01:

1. Downloaded Splunk Universal Forwarder
2. Ran installer
3. Selected: 

```text
On-Premises Splunk Enterprise Instance
```

4. Entered: 
   - Server: `SRV01` IP (e.g., 192.168.56.11)
   - Port: `9997`
5. Completed installation

### Step 3: Enable Receiving on SRV01

On SRV01:

1. Opened Splunk Web
2. Navigated to:

```text
Settings → Forwarding and Receiving
```

3. Selected:

```text
Configure Receiving → New Receiving Port
```

4. Entered:

```text
9997
```

### Step 4: Configure Log Inputs on CLIENT01

Created configuration files in:

```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local
```

#### inputs.conf

```text
[WinEventLog://Application]
disabled = 0
index = main

[WinEventLog://System]
disabled = 0
index = main

[WinEventLog://Security]
disabled = 0
index = main
```

#### outputs.conf

```text
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.56.11:9997
```

### Step 5: Restart Forwarder

```cmd
net stop splunkforwarder
net start splunkforwarder
```

### Step 6: Verify Log Ingestion

On SRV01:

```spl
index=* | stats count by host, sourcetype
```

Observed:

- Logs from both:
  - SRV01
  - CLIENT01

> ⚠️ Initial configuration issues prevented logs from being ingested from CLIENT01. This was resolved by recreating configuration files with proper formatting and file extensions.

> See [Troubleshooting: Splunk Forwarder Log Ingestion Issues](../../troubleshooting/splunk-forwarder-issues.md)

### Step 7: Validate Security Events

```spl
index=main host=CLIENT01 sourcetype=WinEventLog:Security
```

Observed:

- EventCode 4624 (successful login)
- EventCode 4625 (failed login)

### Step 8: Correlate Across Systems

```spl
index=main (EventCode=4624 OR EventCode=4625) 
| stats count by host, EventCode
```

> 💡 This demonstrates centralized visibility across multiple systems.

### Conclusion

The Splunk Universal Forwarder was successfully configured on CLIENT01, enabling centralized log collection into Splunk on SRV01. Windows Event Logs were ingested and made searchable across the environment.

This lab demonstrates the importance of centralized logging in security operations, allowing analysts to monitor and correlate activity across multiple systems rather than relying on isolated local logs.

This setup forms the foundation for detection engineering, threat hunting, and SIEM-based monitoring.