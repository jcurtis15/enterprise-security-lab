# Splunk Introduction and Log Ingestion

## Overview

This lab introduces Splunk as a Security Information and Event Management (SIEM) platform for centralized log collection and analysis. Splunk was installed on SRV01, configured to monitor local Windows Event Logs, and used to search authentication-related events.

## Objective

- Install and access Splunk Enterprise
- Configure monitoring for local Windows Event Logs
- Ingest Security, System, and Application logs
- Perform basic searches in Splunk
- Identify authentication events using Splunk queries

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01
- Client Machine: CLIENT01
- Network: LAB-NET (Internal Network)
- SIEM Platform: Splunk Enterprise
- Host System for Splunk: SRV01

## Steps Performed

### Step 1: Prepare Internet Access on SRV01

SRV01 was originally configured only on the internal lab network and did not have internet access.

To allow Splunk download and installation:

1. Powered off SRV01
2. Opened **VirtualBox** → **Settings** → **Network**
3. Verified:
   - **Adapter 1** remained attached to **Internal Network** (`LAB-NET`)
4. Enabled **Adapter 2**
5. Set **Adapter 2** to:

```text
Attached to: NAT
```

6. Started SRV01
7. Verified connectivity using:

```cmd
ipconfig
ping 8.8.8.8
```

> 💡 Adapter 1 maintained lab communication, while Adapter 2 provided outbound internet access through NAT.

### Step 2: Download and Install Splunk

On SRV01:

1. Opened a web browser
2. Navigated to the Splunk Enterprise download page
3. Downloaded the installer
4. Ran the installer
5. Accepted the license agreement
6. Completed installation
7. Created Splunk administrator credentials during setup

### Step 3: Access Splunk Web Interface

After installation:

1. Opened a web browser on SRV01
2. Navigated to:

```text
http://localhost:8000
```

3. Logged in using the Splunk administrator account created during installation

### Step 4: Add Local Windows Event Logs

In Splunk:

1. Navigated to:

```text
Settings → Add Data
```

2. Selected:

```text
Monitor
```

3. Chose:

```text
Local Event Logs
```

4. Selected the following logs for monitoring:
   - Application
   - System
   - Security
5. Left the **Host** field set to:

```text
SRV01
```

6. Completed the data input configuration and submitted the changes

> 💡 The **Monitor** option enables continuous ingestion of live event log data.
>
> 💡 The **Host** field identifies which system the logs originate from and is useful for future searching and filtering.

### Step 5: Verify Data Ingestion

In Splunk, opened:

```text
Search & Reporting
```

Ran the following search:

```spl
index=main
```

Result:

- Windows event log data was successfully ingested and displayed in Splunk

> ⚠️ At this stage, Splunk is only ingesting logs from SRV01. Logs from other systems (DC01, CLIENT01) are not yet centralized and will be integrated in later labs.

### Step 6: Search Authentication Events

To identify failed login attempts, ran:

```spl
index=main EventCode=4625
```

To identify successful login attempts, ran:

```spl
index=main EventCode=4624
```

Result:

- Authentication events were successfully returned from the Security log

> 💡 EventCode 4624 represents successful logins, while EventCode 4625 represents failed login attempts.

### Step 7: Correlate Authentication Events

To compare authentication activity by event type and account, ran:

```spl
index=main (EventCode=4624 OR EventCode=4625) 
| stats count by EventCode, Account_Name
```

Result:

- Splunk summarized counts of successful and failed login events by account

### Step 8: Visualize Authentication Timeline

To view authentication activity over time, ran:

```spl
index=main (EventCode=4624 OR EventCode=4625) 
| timechart count by EventCode
```

Result:

- Authentication events were visualized over time
- Failed and successful login activity could be analyzed more efficiently than in Event Viewer

> 💡 SIEM platforms allow analysts to aggregate, search, and visualize large volumes of log data, enabling faster detection of suspicious patterns compared to manual log review.

### Conclusion

Splunk was successfully installed and configured on SRV01 to ingest local Windows Event Logs. Security, System, and Application logs were monitored, and authentication-related events were queried using basic Splunk searches.

This lab demonstrates the shift from manual log analysis in Event Viewer to centralized log collection and query-based analysis in a SIEM platform. Splunk provides improved visibility, faster searching, and better support for identifying authentication trends and suspicious behavior.

This lab establishes the foundation for future detection engineering, dashboard creation, and threat hunting using Splunk, where centralized visibility across systems enables detection of patterns that may not be visible when analyzing logs on a single machine.

