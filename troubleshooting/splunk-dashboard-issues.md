# Troubleshooting Splunk Dashboard and Authentication Monitoring Issues

## Overview

This document outlines issues encountered while building and validating the Splunk authentication monitoring dashboard. These issues impacted log visibility, dashboard accuracy, and detection effectiveness.

> ⚠️ These issues highlight common challenges encountered when building SIEM dashboards in a lab environment.

## Scope

This document focuses on:

- Missing or delayed log data
- Dashboard configuration issues
- Time synchronization problems
- Field inconsistencies in Windows logs

## Issue 1: Logs Not Appearing in Dashboard

### Problem

Authentication activity was generated on CLIENT01, but no updates were visible in the dashboard.

### Cause

Dashboard panels were not linked to a shared time picker, causing them to use static or incorrect time ranges.

### Resolution

- Added a time input to the dashboard
- Configured panels to use:

`Time Range → Shared Time Picker`

### Key Takeaway

> Dashboards require explicit time configuration. Without a shared time picker, panels may not reflect current data.

## Issue 2: Logs Missing or Inconsistent in Splunk

### Problem

New authentication events were not appearing or appeared incomplete.

### Cause

Time zone mismatch between systems:

- CLIENT01: Pacific Time
- SRV01: Eastern Time

This caused events to fall outside the dashboard and search time windows, making recent activity appear missing.

### Impact

- Detection logic failed to trigger correctly
- Dashboard visualizations appeared incomplete
- Recent authentication activity was not visible

### Resolution

- Standardized time zone across systems
- Resynced system time

```cmd
w32tm /resync
```

### Key Takeaway

> SIEM systems rely on consistent time across all systems. Time discrepancies can cause missing logs and inaccurate detection.

## Issue 3: Missing Successful Login Events (4624)

### Problem

Failed logins were visible, but successful logins were missing.

### Cause

Events appeared outside expected search time ranges due to time misalignment.

### Resolution

- Corrected system time
- Adjusted dashboard time range
- Verified ingestion using `_time` and `_indextime`

### Key Takeaway

> Always verify both `_time` and `_indextime` when troubleshooting ingestion issues.

## Issue 4: Inconsistent User Field Values

### Problem

Some queries failed to display expected users or showed blank values.

### Cause

Windows logs store usernames in multiple fields:

- `Account_Name`
- `TargetUserName`

### Resolution

Normalized user field:

```spl
| eval User=coalesce(TargetUserName, Account_Name)
```

### Key Takeaway

> Field normalization is critical when working with Windows Event Logs.

## Issue 5: Dashboard Studio UI Confusion

### Problem

Difficulty locating options such as visualization and time configuration.

### Cause

Differences between Splunk Dashboard Studio and Classic Dashboard UI.

### Resolution

- Used "Statistics Table" to build panels
- Converted to visualization after running queries
- Configured inputs through Dashboard Studio controls

### Key Takeaway

> Splunk UI variations can impact workflow, but underlying functionality remains consistent.

## Summary

Issues encountered during this lab were caused by a combination of:

- Time synchronization problems
- Dashboard configuration issues
- Field inconsistencies in logs
- UI differences within Splunk

After resolving these issues:

- Dashboard displayed real-time data accurately
- Detection logic functioned correctly
- Visualizations reflected actual system activity

This troubleshooting process highlights the importance of validating:

- Time synchronization
- Data ingestion
- Dashboard configuration
- Field consistency

when building SIEM dashboards.

> 💡 In SIEM environments, visibility issues are often caused by configuration or time-related discrepancies rather than data absence.