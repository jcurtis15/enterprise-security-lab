# Splunk Brute Force Detection Lab

## Overview

This lab demonstrates how to detect brute force login activity using Splunk by analyzing Windows Security Event logs.

Using logs collected from CLIENT01, multiple failed login attempts (Event ID 4625) were correlated with successful logins (Event ID 4624) to identify potential account compromise scenarios.

This lab builds on the centralized logging pipeline established in previous labs and introduces detection engineering concepts within a SIEM.

## Objectives

- Generate brute force login activity on CLIENT01
- Identify failed and successful login events in Splunk
- Correlate Event ID 4625 and 4624
- Build detection logic using Splunk Processing Language (SPL)
- Apply time-based analysis to simulate real attack behavior

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01 (Splunk Enterprise)
- Client Machine: CLIENT01 (Windows 11, domain joined)
- Network: LAB-NET (Internal) + NAT (Internet access)

## Steps Performed

### Step 1: Simulate Brute Force Activity

On CLIENT01:

1. Locked or logged out of the system
2. Attempted login using:

```text
CORP\alice
```

3. Entered incorrect password multiple times (5-10 attempts)
4. Successfully logged in after failed attempts

> 💡 This creates a realistic brute force pattern:
>
> Multiple failures followed by a successful login.

### Step 2: Verify Event Logs in Splunk

On SRV01:

```spl
index=* host=CLIENT01 (EventCode=4624 OR EventCode=4625)
| table _time, EventCode, Account_Name, Source_Network_Address
| sort - _time
```

Observed:

- EventCode 4625 → Failed login attempts
- EventCode 4624 → Successful login

### Step 3: Analyze Failed Login Activity

```spl
index=* EventCode=4625
| stats count by Account_Name, Source_Network_Address
| sort - count
```

> 💡 Identifies accounts experiencing repeated login failures.

### Step 4: Apply Threshold for Detection

```spl
index=* EventCode=4625
| stats count by Account_Name, Source_Network_Address
| where count >= 5
```

> ⚠️ Threshold-based detection helps reduce noise and identify suspicious behavior

### Step 5: Correlate Failed and Successful Logins

```spl
index=* (EventCode=4625 OR EventCode=4624)
| stats 
    count(eval(EventCode=4625)) as failed_attempts,
    count(eval(EventCode=4624)) as successful_logins
    by Account_Name, Source_Network_Address
| where failed_attempts >= 5 AND successful_logins >= 1
```

Observed:

- Accounts with repeated failures followed by success
- Indicates potential brute force compromise

### Step 6: Add Time-Based Detection

```spl
index=* (EventCode=4625 OR EventCode=4624)
| bin _time span=5m
| stats 
    count(eval(EventCode=4625)) as failed_attempts,
    count(eval(EventCode=4624)) as successful_logins
    by _time, Account_Name, Source_Network_Address
| where failed_attempts >= 5 AND successful_logins >= 1
```

> 💡 This limits detection to activity within a 5-minute window, simulating real attack timing.

### Step 7: Format Detection Output

```spl
index=* (EventCode=4625 OR EventCode=4624)
| bin _time span=5m
| stats 
    count(eval(EventCode=4625)) as failed_attempts,
    count(eval(EventCode=4624)) as successful_logins
    by _time, Account_Name, Source_Network_Address
| where failed_attempts >= 5 AND successful_logins >= 1
| rename Account_Name as User, Source_Network_Address as Source_IP
| sort - _time
```

### Detection Logic Summary

Brute force activity was identified based on the following conditions:

- 5 or more failed login attempts (Event ID 4625)
- At least 1 successful login (Event ID 4624)
- Occurring within a 5-minute time window
- Matching user and source

This logic simulates a common attack pattern where an attacker successfully guesses credentials after multiple failed attempts.

### Challenges Encountered

During this lab, several issues were encountered that impacted log ingestion and detection accuracy:

- Logs from CLIENT01 were not appearing in Splunk in real time
- Network connectivity to the Splunk receiving port (9997) was initially blocked 
- The Splunk Universal Forwarder required a restart after resolving connectivity issues

These issues prevented accurate correlation of login events and delayed detection results.

> ⚠️ Detection accuracy depends heavily on reliable log ingestion. Issues at the data collection or transport layer can directly impact visibility and analysis.

> See [Troubleshooting: Splunk Brute Force Detection Issues](../../troubleshooting/splunk-brute-force-issues.md)

### Conclusion

This lab demonstrates how brute force attacks can be detected through centralized log analysis in Splunk.

By correlating failed and successful login events and applying thresholds and time-based logic, it is possible to identify suspicious authentication behavior indicative of compromise.

This lab introduces key SIEM concepts, including:

- Event correlation
- Threshold-based detection
- Time-based analysis
- Detection engineering using SPL

This detection forms the foundation for alerting, dashboards, and more advanced threat detection use cases.