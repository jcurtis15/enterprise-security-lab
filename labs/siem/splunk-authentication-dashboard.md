# Splunk Authentication Monitoring Dashboard

## Overview

This lab demonstrates how to build a centralized authentication monitoring dashboard using Splunk. The dashboard visualizes Windows Security Event logs to provide real-time visibility into login activity, failed authentication attempts, and potential brute force behavior.

This lab builds on previous work involving log ingestion, detection logic, and alerting, and introduces visualization as a critical component of a SIEM workflow.

## Objective

- Build a Splunk dashboard for authentication monitoring
- Visualize successful and failed login activity
- Identify targeted users and login sources
- Display logon type distribution
- Integrate brute force detection into dashboard view
- Validate dashboard accuracy using simulated user activity

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01 (Splunk Enterprise)
- Client Machine: CLIENT01 (Windows 11, domain-joined)
- Network: LAB-NET (Internal) + NAT (Internet access)

## Steps Performed

### Step 1: Create Dashboard

On SRV01:

1. Open Splunk Web
2. Navigate to:

```text
Search & Reporting → Dashboards → Create New Dashboard
```

3. Configure:
   - Title: Authentication Monitoring
   - Dashboard Studio enabled

### Step 2: Add Time Picker

1. Click:

```text
Add Input → Time
```

2. Configure default time range:

```text
Last 15 minutes
```

3. This allows all panels to dynamically reflect recent activity.

> 💡 Panels must be explicitly linked to the shared time picker in Dashboard Studio.

### Step 3: Build Dashboard Panels

Each panel was created using **New** → **Statistics Table**, then converted to a visualization.

The following panels were created to provide visibility into authentication activity across the environment:

-----------------

#### Failed Logins Over Time

```spl
index=main host=CLIENT01 EventCode=4625 Logon_Type=2 
| timechart count as failed_logins
```

------------------

#### Successful Logins Over Time

```spl
index=main host=CLIENT01 EventCode=4624 Logon_Type=2
| timechart count as successful_logins
```

-----------------

#### Top Targeted Users

```spl
index=main host=CLIENT01 EventCode=4625 
| eval User=coalesce(TargetUserName, Account_Name) 
| stats count as failed_attempts by User 
| sort - failed_attempts
```

-------------------

#### Login Activity by Source

```spl
index=main host=CLIENT01 (EventCode=4624 OR EventCode=4625) 
| stats 
	count(eval(EventCode=4625)) as failed_attempts, 
	count(eval(EventCode=4624)) as successful_logins 
	by Source_Network_Address 
| sort - failed_attempts
```

----------------------

#### Logon Type Breakdown

```spl
index=main host=CLIENT01 (EventCode=4624 OR EventCode=4625)
| eval Logon_Type_Name=case(
    Logon_Type==2, "Interactive (Local)",
    Logon_Type==10, "Remote (RDP)",
    Logon_Type==3, "Network",
    true(), "Other"
)
| stats count by Logon_Type_Name
| sort - count
```

-------------------------

#### Brute Force Detection Panel

```spl
index=main host=CLIENT01 (EventCode=4625 OR EventCode=4624) 
| eval User=coalesce(TargetUserName, Account_Name) 
| bin _time span=5m 
| stats 
	count(eval(EventCode=4625)) as failed_attempts, 
	count(eval(EventCode=4624)) as successful_logins 
	by _time, User 
| where failed_attempts >= 5 AND successful_logins >= 1 
| sort - _time
```

### Step 4: Link Panels to Time Picker

Each panel was configured to use:

```text
Time Range → Shared Time Picker
```

### Step 5: Validate Dashboard

Simulated authentication activity on CLIENT01:

- Multiple failed login attempts
- Successful logins
- Activity across multiple users (alice and bob)
- Activity spread across time intervals

Observed:

- Failed login spikes reflected in charts
- Successful logins appeared correctly
- Top users panel identified targeted accounts
- Brute force detection triggered appropriately

> 💡 Local logins appear as `127.0.0.1` due to Logon Type 2 behavior.

> 💡 Dashboard accuracy was validated by comparing real-time activity against expected behavior, ensuring that visualizations correctly reflected underlying log data.

## Key Concepts

- **Visualization Layer**: Dashboards provide situational awareness, not raw data
- **Field Normalization**: `coalesce()` ensures consistent user identification
- **Time Dependency**: Accurate time configuration is critical for SIEM functionality
- **Detection vs Visualization**: Detection logic feeds dashboards, but dashboards require proper configuration

## Challenges Encountered

- Time zone mismatch caused events to fall outside search windows
- Dashboard panels initially did not reflect data due to missing time picker configuration
- Field inconsistencies required normalization using `coalesce()`
- Splunk Dashboard Studio UI differences required adjustment in workflow

> ⚠️ Issues encountered during this lab are documented in [Troubleshooting: Splunk Dashboard Issues](../../troubleshooting/splunk-dashboard-issues.md)

### Conclusion

This lab demonstrates how authentication data can be transformed into actionable visual insights using Splunk dashboards.

By building multiple panels and validating them with simulated activity, centralized visibility into authentication behavior was achieved. This enables analysts to quickly identify anomalies such as brute force attempts and targeted user attacks.

This lab completes the SIEM workflow by adding a visualization layer to previously implemented logging, detection, and alerting components.