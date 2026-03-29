# Troubleshooting Splunk Brute Force Detection and Log Ingestion Issues

## Overview

This document outlines issues encountered during the brute force detection lab where logs from CLIENT01 were delayed, missing, or not updating in Splunk on SRV01.

These issues impacted the ability to perform real-time detection and correlation of login activity.

> ⚠️ These issues highlight how failures in log ingestion and transport can directly impact detection accuracy in SIEM environments.

## Scope

This document focuses on issues affecting:

- Splunk Universal Forwarder connectivity
- Windows Firewall blocking Splunk traffic
- Forwarder behavior after connection failures
- Log ingestion delays and missing data

## Issue 1: No Recent Logs Appearing in Splunk

### Problem

Logs from CLIENT01 were visible in Splunk, but no new events were being ingested. The most recent logs were outdated, preventing real-time analysis.

### Cause

Network connectivity to SRV01 on port `9997` was blocked, preventing the Splunk Universal Forwarder from sending data.

Testing from CLIENT01:

```powershell
Test-NetConnection 192.168.56.11 -Port 9997
```

Result:

```text
TcpTestSucceeded : False
```

Further investigation using:

```powershell
netstat -ano | findstr 9997
```

Showed:

```text
SYN_SENT
```

This behavior is consistent with traffic being blocked or dropped before reaching the destination service.

### Resolution

The issue was caused by Windows Firewall on SRV01 blocking inbound traffic on port `9997`.

#### Steps

1. On SRV01, created a firewall rule:

```powershell
New-NetFirewallRule -DisplayName "Allow Splunk 9997" -Direction Inbound -Protocol TCP -LocalPort 9997 -Action Allow
```

2. Verified Splunk was listening:

```powershell
netstat -ano | findstr 9997
```

3. Retested connectivity from CLIENT01:

```powershell
Test-NetConnection 192.168.56.11 -Port 9997
```

Result:

```text
TcpTestSucceeded : True
```

#### Key Takeaways

> `SYN_SENT` indicates a connection attempt with no response, typically caused by firewall blocking.

> Splunk may be listening on a port, but Windows Firewall can still prevent access.

## Issue 2: Forwarder Not Sending Logs After Connectivity Fix

### Problem

After restoring network connectivity, logs from CLIENT01 still did not appear in Splunk.

### Cause

The Splunk Universal Forwarder remained in a failed or stalled state after repeated connection failures.

### Resolution

Restarted the Splunk Universal Forwarder service on CLIENT01:

```powershell
Restart-Service SplunkForwarder
```

After restarting, log ingestion resumed and new events began appearing in Splunk.

#### Key Takeaway

> The Splunk Universal Forwarder may require a restart after resolving connectivity issues.

## Issue 3: Incorrect CLI Usage When Managing Forwarder

### Problem

Attempting to navigate to the forwarder directory using Command Prompt resulted in:

```text
The system cannot find the path specified
```

### Cause

Command Prompt does not switch drives when using cd without the /d flag.

### Resolution

Used PowerShell instead:

```powershell
Set-Location "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe list forward-server
```

Alternatively, in Command Prompt:

```cmd
cd /d "C:\Program Files\SplunkUniversalForwarder\bin"
```

#### Key Takeaways

> PowerShell handles paths with spaces more reliably.

> In Command Prompt, use `cd /d` when changing drives.

## Issue 4: Detection Not Updating Due to Ingestion Delay

### Problem

Brute force detection queries were not returning expected results immediately after generating log events.

### Cause

Log ingestion was delayed due to previous connectivity issues and forwarder instability.

### Resolution

Validated ingestion timing using:

```spl
index=*
| eval delay=_indextime - _time
| table _time, _indextime, delay, host, EventCode
| sort - _indextime
```

Confirmed that:

- `_indextime` reflected current ingestion
- Delay was minimal after fixes were applied

#### Key Takeaway

> Comparing `_time` and `_indextime` helps distinguish between ingestion delays and timestamp issues.

## Summary

Log ingestion and detection issues were caused by a combination of:

- Windows Firewall blocking port `9997`
- Forwarder requiring restart after connection failures
- Incorrect command-line usage when managing the forwarder
- Temporary ingestion delays following reconnection

After resolving these issues:

- CLIENT01 successfully forwarded logs to SRV01
- Logs were ingested in near real-time
- Brute force detection queries returned accurate results

This troubleshooting process highlights the importance of validating:

- Network connectivity
- Firewall rules
- Forwarder health
- Log ingestion timing

when diagnosing SIEM data pipeline issues.