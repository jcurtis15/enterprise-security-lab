# Windows Event Logging

## Overview

This lab introduces Windows Event Logging and explores how system and security events are recorded and analyzed within a domain environment. Event Viewer was used to examine different log types, identify authentication events, and filter logs for analysis.

## Objective

- Understand Windows Event Logs and their purpose
- Identify key log categories (Application, System, Security)
- Analyze authentication-related events
- Filter and inspect log entries
- Validate access to Security logs

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01
- Client Machine: CLIENT01
- Network: LAB-NET (Internal Network)

## Steps Performed

### Step 1: Open Event Viewer

On CLIENT01:

1. Click **Start**
2. Search for:

```text
Event Viewer
```

3. Open the application

### Step 2: Explore Windows Logs

In the left pane:

```text
Event Viewer → Windows Logs
```

The following log categories were reviewed:

- Application
- System
- Security

> 💡 The Security log contains authentication and auditing events and is critical for security monitoring.

### Step 3: Access Security Logs

Navigated to:

```text
Windows Logs → Security
```

Observed:

- Event ID
- Date and Time
- User Account
- Source

### Step 4: Identify Logon Events

Filtered for successful login events:

```text
Event ID: 4624
```

Result:

- Successful authentication events were identified
- Event details included account name, logon type, and source

### Step 5: Identify Failed Logins

Filtered for failed login events:

```text
Event ID: 4625
```

Result:

- Failed authentication attempts were visible (if present)

### Step 6: Filter Log Data

Used **Filter Current Log** to isolate events:

- 4624 (successful logins)
- 4625 (failed logins)

> 💡 Filtering logs is a critical skill for analyzing large datasets in real-world environments.

### Step 7: Analyze Event Details

Selected individual events and reviewed:

- Account Name
- Domain
- Logon Type
- Workstation Name

> 💡 Logon types help determine how access occurred (interactive, network, remote, etc.).

> 💡 Security event logs are a primary source of evidence during incident investigations and are widely used in security monitoring and SIEM platforms.

### Step 8: Validate Security Log Access

Initial attempts to access the Security log using the `itadmin` account resulted in an **Access Denied** error.

Access was successfully obtained by:

- Logging in as `CORP\administrator`
- Adding `corp\itadmin` to the local **Event Log Readers** group on CLIENT01

```cmd
net localgroup "Event Log Readers" corp\itadmin /add
```

After logging out and back in, the `itadmin` account was able to access Security logs.

> ⚠️ Access to the Security log was initially denied for the itadmin account due to permission restrictions.

> See [Troubleshooting: Security Log Access Issues](../../troubleshooting/event-log-access-issues.md)

### Conclusion

Windows Event Logging was successfully explored using Event Viewer. Key log categories were identified, and authentication events were analyzed using event IDs and filtering techniques.

A permissions-related issue was encountered when accessing Security logs, highlighting the importance of proper privilege configuration. This lab demonstrated how log access is controlled and reinforced the role of event logs in monitoring system activity and detecting potential security incidents.

Understanding and analyzing event logs is a foundational skill in both system administration and cybersecurity operations and serves as a basis for centralized logging and future SIEM integration, where events are aggregated, analyzed, and used for threat detection.