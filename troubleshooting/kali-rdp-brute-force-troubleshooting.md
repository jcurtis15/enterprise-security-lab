# Troubleshooting Kali Brute Force Attack and RDP Connectivity Issues

## Overview

This document outlines issues encountered while configuring Kali Linux, establishing RDP connectivity, and executing brute force attacks against CLIENT01.

## Scope

- Kali networking and routing issues
- DNS resolution failures
- RDP authentication and authorization problems
- Hydra attack execution issues

## Impact

- Prevented initial attack execution
- Delayed validation of detection logic
- Required multiple layers of troubleshooting (network, DNS, authentication)

## Issue 1: Kali Unable to Reach Internal Network

### Problem

Kali could not communicate with CLIENT01.

### Cause

Incorrect VirtualBox network configuration or missing IP assignment.

### Resolution

- Configured VirtualBox Adapter 1 as Internal Network (`LAB-NET`)
- Verified IP using:

```bash
ip a
```

## Issue 2: No Internet Access on Kali

### Problem

Kali could not access the internet.

### Cause

Incorrect default route:

```text
default via 192.168.56.1 dev eth0
```

### Resolution

Removed incorrect default route:

```bash
sudo ip route del default via 192.168.56.1 dev eth0
```

### Key Takeaway

> Systems with multiple interfaces must have a correct default route for internet connectivity.

## Issue 3: DNS Resolution Failure

### Problem

```text
Temporary failure resolving 'http.kali.org'
```

### Cause

Missing or misconfigured DNS server.

### Resolution

Unlocked and modified `/etc/resolv.conf`:

```bash
sudo chattr -i /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
```

### Key Takeaway

> DNS issues can prevent package installation even when network connectivity is functional.

## Issue 4: RDP Connection Failure (Kerberos / NLA)

### Problem

```text
Cannot find KDC for realm "CORP"
ERRCONNECT_LOGON_FAILURE
```

### Cause

Kali was attempting Kerberos authentication without proper domain configuration.

### Resolution

Forced NTLM / RDP security:

```bash
xfreerdp /v:192.168.56.20 /u:'CORP\bob' /p:'Password1234!' /sec:rdp /cert:ignore
```

Disabled NLA on CLIENT01.

### Key Takeaway

> Kerberos authentication requires domain connectivity. NTLM is often required in lab environments.

## Issue 5: RDP Authorization Failure

### Problem

```text
User not allowed to log in via Remote Desktop
```

### Cause

User was not part of Remote Desktop Users group.

### Resolution

```powershell
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "CORP\bob"
```

### Key Takeaway

> Authentication success does not guarantee authorization. Access control policies must be configured correctly.

## Issue 6: Hydra Connection Failures

### Problem

```text
freerdp: The connection failed to establish
```

### Cause

RDP sensitivity to multiple parallel connections.

### Resolution

Reduced Hydra concurrency:

```bash
hydra -t 1 -W 1 -L users.txt -P passwords.txt rdp://192.168.56.20
```

### Key Takeaway

> Some protocols (like RDP) require reduced concurrency during brute force testing.

## Summary

Issues encountered were related to:

- Network routing and configuration
- DNS resolution
- Authentication method mismatches
- Authorization permissions
- Tool-specific limitations

> 💡 These issues reflect real-world challenges where connectivity, authentication, and system configuration must all align for successful attack simulation and detection.

After resolving these issues:

- RDP connectivity was established
- Hydra successfully executed brute force attack
- Logs were ingested into Splunk
- Detection logic functioned as expected

> 💡 Real-world attack simulation often requires resolving multiple layers of infrastructure, authentication, and configuration issues.