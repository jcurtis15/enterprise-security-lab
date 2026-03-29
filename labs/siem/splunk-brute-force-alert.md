# Splunk Brute Force Alerting Lab

## Overview

This lab demonstrates how to operationalize brute force detection logic by converting it into a scheduled alert in Splunk.

Building on previously developed detection queries, an alert was configured to trigger when suspicious authentication behavior was identified. This simulates a real-world SOC workflow where detections generate actionable alerts that analysts can investigate.

This lab also introduces the distinction between detection and investigation workflows, highlighting how alerts identify suspicious patterns while separate queries are used to analyze underlying events.

> ⚠️ This lab demonstrates how detection logic transitions into actionable alerting within a SIEM workflow.

## Objective

- Convert brute force detection logic into a Splunk alert
- Configure scheduled alert execution using Cron syntax
- Define trigger conditions for alert activation
- Generate simulated brute force activity
- Validate alert functionality and results
- Perform investigation using follow-up queries

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01 (Splunk Enterprise)
- Client Machine: CLIENT01 (Windows 11, domain-joined)
- Network: LAB-NET (Internal) + NAT (Internet access)

## Steps Performed

### Step 1: Prepare Detection Query

The following query was used to identify brute force login activity:

```spl
index=* (EventCode=4625 OR EventCode=4624)
| bin _time span=5m
| stats 
    count(eval(EventCode=4625)) as failed_attempts,
    count(eval(EventCode=4624)) as successful_logins,
    values(Logon_Type) as logon_types
    by _time, Account_Name, Source_Network_Address
| where failed_attempts >= 5 AND successful_logins >= 1
```

> 💡 This query detects multiple failed login attempts followed by a successful login within a 5-minute window.

### Step 2: Create Alert in Splunk

On SRV01:

1. Open Splunk Web
2. Navigate to:

```text
Apps → Search & Reporting
```

3. Paste and run the detection query
4. Select:

```text
Save As → Alert
```

### Step 3: Configure Alert Settings

**Basic Information**

- Title: `Brute Force Detection Alert`
- Description: Detects multiple failed logins followed by a successful login

**Alert Type**

```text
Scheduled Alert → Run on Cron Schedule
```

**Schedule**

Since preset schedule options did not allow 5-minute intervals, a Cron schedule was used:

```text
*/5 * * * *
```

> 💡 This runs the alert every 5 minutes.

### Step 4: Configure Trigger Conditions

```text
Trigger alert when: Number of results > 0
```

### Step 5: Configure Trigger Actions

Enabled:

```text
Add to Triggered Alerts
```

> ⚠️ At least one trigger action must be configured for alerts to generate visible results.

### Step 6: Generate Test Activity

On CLIENT01:

1. Attempt login using:

```text
CORP\alice
```

2. Enter incorrect password multiple times (5-10 times)
3. Successfully log in

### Step 7: Validate Alert Trigger

On SRV01:

Navigate to:

```text
Activity → Triggered Alerts
```

Observed:

- Alert triggered successfully
- Execution time aligned with test activity
- Results included:
  - Account_Name
  - Source_Network_Address
  - failed_attempts
  - successful_logins

> 💡 Alerts trigger based on scheduled execution, not immediately upon event generation. Timing depends on the configured schedule interval.

### Step 8: Investigate Alert Results

After the alert triggered, additional queries were used to review underlying events.

```spl
index=* host=CLIENT01 Account_Name="alice" (EventCode=4624 OR EventCode=4625)
| sort - _time
| table _time, EventCode, Account_Name, Logon_Type, Source_Network_Address
```

> ⚠️ Aggregated detection queries using `stats` do not retain raw event data. A separate investigation query is required to analyze underlying authentication events.

Observed:

- Multiple failed login attempts (EventCode=4625)
- A successful login (EventCode=4624)
- Logon Type = 2 (interactive login)
- Source_Network_Address = 127.0.0.1 (local login)

> 💡 Local logins (Logon Type 2) may display `127.0.0.1` instead of a real network IP address

### Detection Logic Summary

This alert detects brute force login behavior by correlating failed and successful authentication events within a defined time window.

Conditions:

- 5 or more failed login attempts (EventCode=4625)
- At least 1 successful login (EventCode=4624)
- Occurring within a 5-minute time window
- Matching user and source

### Challenges Encountered

During this lab, several challenges impacted alert creation and validation:

- Splunk UI required Cron syntax to configure 5-minute scheduling intervals
- Alert configuration options varied depending on interface layout
- Aggregated queries using `stats` did not display raw events when using "View events"
- Local authentication events displayed `127.0.0.1` instead of a remote IP due to Logon Type 2 behavior

> ⚠️ Detection queries are optimized for performance and may not retain raw event context. Separate investigation queries are required to analyze underlying events.

> ⚠️ Alerting depends on reliable log ingestion. Any disruption in the logging pipeline can prevent alerts from triggering.

> ⚠️ See [Troubleshooting: Splunk Brute Force Detection Issues](../../troubleshooting/splunk-brute-force-issues.md)

### Conclusion

This lab demonstrates how detection logic can be operationalized into actionable alerting within Splunk.

By configuring scheduled alerts and trigger conditions, suspicious authentication activity can be automatically identified and surfaced for investigation.

This lab completes a core SIEM workflow:

- Log ingestion
- Detection engineering
- Alert generation
- Investigation

This foundation enables further development of dashboards, automated response, and advanced threat detection use cases.

> 💡 In a production environment, alerts like this would feed into a SOC workflow for triage, investigation, and potential incident response.

















