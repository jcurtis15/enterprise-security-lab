# Troubleshooting Splunk Forwarder Log Ingestion Issues

## Overview

This document outlines issues encountered when configuring the Splunk Universal Forwarder on CLIENT01. Despite successful installation and network connectivity, logs from CLIENT01 were not appearing in Splunk on SRV01.

## Scope

This document focuses on configuration-related issues affecting log ingestion, including file formatting, file extensions, and forwarder configuration behavior.

## Issue 1: Logs Not Appearing in Splunk

### Problem

When running the following Splunk search on SRV01:

```spl
index=* | stats count by host
```

Only logs from SRV01 were present. Logs from CLIENT01 were not visible.

### Cause

The Splunk Universal Forwarder on CLIENT01 was not properly loading configuration files due to the following issues:

- `inputs.conf` was saved as:

```text
inputs.conf.txt
```

- Improper file extension prevented Splunk from reading the configuration
- Configuration file contents were malformed due to incorrect command-line creation
- Splunk logs indicated:

```text
No stanzas found for scheme 'WinEventLog'
```

As a result, no Windows Event Logs were being monitored or forwarded.

This prevented CLIENT01 from sending any data to SRV01.

### Resolution

The issue was resolved by deleting the incorrectly created configuration files and recreating them using Notepad with proper formatting and file extensions.

#### Steps:

1. Navigate to:

```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local
```

2. Delete invalid files:

```text
inputs.conf.txt
outputs.conf.txt
```

3. Open Notepad as Administrator
4. Create `inputs.conf`:

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

5. Create `outputs.conf`:

```text
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.56.11:9997
```

6. Save both files with:
   - File type: **All Files (*.*)**
   - Correct `.conf` extension
7. Restart the forwarder:

```cmd
net stop splunkforwarder
net start splunkforwarder
```

8. Generate new events (logon/logoff or failed login attempts)

After completing these steps, logs from CLIENT01 began appearing in Splunk.

#### Key Takeaways

> Splunk Universal Forwarder only reads `.conf` files. Incorrect file extensions will prevent configuration from being loaded.

> Improperly formatted configuration files can prevent Splunk from recognizing inputs and forwarding data.

> Splunk logs can provide valuable insight into configuration issues, such as missing or invalid stanzas.

## Issue 2: Configuration Appeared Correct but Still Not Working

### Problem

Even after creating configuration files, logs were still not appearing in Splunk.

### Cause

Configuration files were initially created using command-line `echo` commands, which resulted in malformed file contents. The files contained unintended text (such as literal `echo` statements), resulting in invalid configuration syntax.

### Resolution

Configuration files were recreated using Notepad to ensure proper formatting and structure.

#### Key Takeaway

> Configuration files should be created or verified using a text editor to avoid formatting issues introduced by command-line methods.

> Even when services and network connectivity are functioning correctly, misconfigured inputs will prevent data from being ingested.

### Summary

Log ingestion issues were caused by improperly created and formatted configuration files that were not recognized by the Splunk Universal Forwarder. Specifically, incorrect file extensions and malformed content prevented it from loading inputs and forwarding logs.

Recreating the configuration files with proper formatting and correct `.conf` extensions resolved the issue, allowing logs from CLIENT01 to be successfully ingested into Splunk.

This troubleshooting process highlights the importance of validating configuration files and understanding how Splunk processes inputs and outputs when diagnosing log ingestion issues.