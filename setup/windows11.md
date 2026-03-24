# Windows 11 Client Setup (CLIENT01)

## Overview

This document outlines the installation and configuration of a Windows 11 client (CLIENT01) within an Oracle VirtualBox lab environment. The system is configured to join the domain and simulate a typical end-user workstation.

## Objective

- Install Windows 11 in a virtual environment
- Configure system settings for lab use
- Set static IP and DNS
- Join the client to the domain
- Validate domain connectivity and user authentication

## Environment

- VM Name: CLIENT01
- OS: Windows 11 Enterprise
- Static IP: 192.168.56.20
- Subnet: 255.255.255.0
- DNS: 192.168.56.10
- Domain: corp.local

## Steps

### Step 1: Virtual Machine Configuration

1. Created a new VM in VirtualBox:
   - Name: `CLIENT01`
   - Type: Microsoft Windows
   - Version: Windows 11 Enterprise (64-bit)
2. Allocated Resources:
   - Memory: 4 GB
   - CPU: 2 cores
   - Disk: 60 GB (dynamically allocated)
3. Configured Network:
   - Adapter 1:
     - Attached to: **Internal Network**
     - Name: `LAB-NET`
     - Cable Connected: Enabled

### Step 2: Install Operating System

1. Mounted Windows 11 ISO

2. Started VM and booted from optical media

3. Selected:

   ```text
   Windows 11 Enterprise
   ```

4. Proceeded through installation prompts

### Step 3: Bypass Windows 11 Hardware Requirements

Due to VirtualBox limitations, Windows 11 hardware checks were bypassed.

1. Pressed: 

```text
Shift + F10
```

2. Opened Registry Editor:

```cmd
regedit
```

3. Navigated to:

```text
HKEY_LOCAL_MACHINE\SYSTEM\Setup
```

4. Created new key:

```text
LabConfig
```

5. Added the following DWORD (32-bit) values:

```text
BypassTPMCheck = 1
BypassSecureBootCheck = 1
BypassRAMCheck = 1
```

6. Closed Registry Editor and Command Prompt to continue installation

### Step 4: Disk Partitioning

1. At the disk selection screen:
   - Multiple partitions were present (EFI, MSR, Primary)
2. Selected the **largest partition labeled "Primary."**
3. Proceeded with installation on the selected partition

> 💡 Windows automatically manages required system partitions during installation
>
> 💡 The Primary partition is used for the Windows installation, while smaller system partitions (EFI, MSR) support boot and system functions.

### Step 5: Complete Installation

1. Continued installation process
2. Allowed the system to reboot as needed
3. Completed initial setup

### Step 6: Initial Setup (Local Account)

1. Selected:

```text
I don't have internet
```

2. Chose:

```text
Continue with limited setup
```

3. Created local account

```text
Username: localadmin
Password: (user-defined)
```

### Step 7: Rename Computer

1. Navigate to:
   - Settings → System → About → Rename this PC
2. Set hostname:

```text
CLIENT01
```

3. Rebooted system

### Step 8: Configure Network Settings

1. Opened Network Connections:

```text
ncpa.cpl
```

2. Right-click on `Ethernet`
3. Click `Internet Protocol Version 4 (TCP/IPv4)` → `Properties`
4. Configure IPv4 settings:

| Field      | Value           |
| ---------- | --------------- |
| IP Address | 192.168.56.20   |
| Subnet     | 255.255.255.0   |
| Gateway    | *(leave blank)* |
| DNS        | 192.168.56.10   |

> ⚠️ DNS must point to the Domain Controller

### Step 9: Verify Network Connectivity

Run the following:

```cmd
ping 192.168.56.10
nslookup corp.local
```

**Expected Results**

```text
Reply from 192.168.56.10

Server: dc01.corp.local
Address: 192.168.56.10
```

### Step 10: Join Domain

1. Navigate to:
   - Settings → System → About → Domain or Workgroup
2. Locate `To rename this computer or change its domain or workgroup, click Change` and click `Change`.
3. Click `Domain` and enter:

```text
corp.local
```

4. Provide domain credentials when prompted:

```text
corp\administrator
```

5. A welcome message should appear confirming the system has joined the domain
6. Restarted system

### Step 11: Verify Domain Join

1. Logged in as:

```text
corp\alice
```

2. Verified user context:

```cmd
whoami
```

**Expected Result**

```text
corp\alice
```

### Step 12: Additional Validation

Confirm domain communication:

```cmd
echo %USERDOMAIN%
```

**Expected Result**

```text
CORP
```

Verify domain controller discovery:

```cmd
nltest /dsgetdc:corp.local
```

**Expected Result**

```text
DC: \\DC01
Address: \\192.168.56.10
```

### Step 13: Snapshot

A snapshot was taken after completing the configuration:

```text
CLIENT01 - Domain Joined
```

This snapshot provides a stable restore point for future lab activities.

### Conclusion

CLIENT01 is now fully configured as a domain-joined Windows 11 client. This system simulates an end-user workstation and will be used for authentication testing, activity generation, and future cybersecurity lab scenarios.

