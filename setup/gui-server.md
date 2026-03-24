# GUI Server Setup (Windows Server 2025 Desktop Experience)

## Overview

This document outlines the setup of a Windows Server 2025 GUI-based server (SRV01) inside Oracle VirtualBox. This server is joined to the domain and configured with administrative tools to manage Active Directory.

## Objective

- Install Windows Server 2025 Desktop Experience
- Configure static IP and DNS
- Join the server to the domain
- Install Remote Server Administration Tools (RSAT)
- Manage Active Directory from a GUI-based system

## Environment

- VM Name: SRV01
- OS: Windows Server 2025 Desktop Experience
- Static IP: 192.168.56.11
- Subnet: 255.255.255.0
- DNS: 192.168.56.10
- Domain: corp.local

## Steps

### Step 1: Virtual Machine Configuration

1. Created a new VM in VirtualBox:
   - Name: `SRV01`
   - Type: Windows
   - Version: Windows Server (64-bit)
2. Allocated resources:
   - Memory: 8 GB
   - CPU: 3 cores
   - Disk: 80 GB (dynamically allocated)
3. Configured network:
   - Adapter 1:
     - Attached to: **Internal Network**
     - Name: `LAB-NET`
     - Cable Connected: Enabled

### Step 2: Install Operating System

1. Mounted Windows Server 2025 ISO
2. Started VM and selected:

```text
Windows Server 2025 Standard Evaluation (Desktop Experience)
```

3. Completed installation with default settings

> ⚠️ If disk partition errors occur, see [Installation Troubleshooting](../troubleshooting/dc-installation.md)

### Step 3: Initial Configuration

**Rename Computer**

1. Navigate to:
   - Settings → System → About → Rename this PC
2. Set hostname:

```text
SRV01
```

3. Reboot the system

### Step 4: Configure Network Settings

**Assign Static IP**

1. Open Network Connections (ncpa.cpl)

2. Right-click **Ethernet** → Properties

3. Configure IPv4

   | Field       | Value         |
   | ----------- | ------------- |
   | IP Address  | 192.168.56.11 |
   | Subnet Mask | 255.255.255.0 |
   | Gateway     | *leave blank* |
   | DNS         | 192.168.56.10 |

> ⚠️ DNS must point to the Domain Controller for domain functionality

### Step 5: Verify Network Connectivity

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

### Step 6: Join Domain

1. Navigate to:
   - System → Rename this PC → Domain
2. Enter:

```text
corp.local
```

3. When prompted, provide credentials:

```text
corp\administrator
```

4. Restart the server

### Step 7: Verify Domain Join

1. Log in using:

```text
corp\administrator
```

2. Verify:

```cmd
whoami
```

**Expected Result**

```text
corp\administrator
```

### Step 8: Install Remote Server Administration Tools (RSAT)

1. Open:
   - Server Manager → Manage → Add Roles and Features
2. Installation Type:
   - Role-based or feature-based installation
3. Server Selection:
   - Select `SRV01`
4. Server Roles:
   - No changes (leave default)
5. Features:
   - Expand:
     - Remote Server Administration Tools
       - Role Administration Tools
6. Select:
   - AD DS and AD LDS Tools
   - Group Policy Management
   - DNS Server Tools (optional but recommended)
7. Install and complete the installation process.

### Step 9: Validate Administrative Tools

Open the following tools from the Start Menu:

- Start → All → Windows Tools
  - Active Directory Users and Computers
  - Group Policy Management
  - DNS Manager (if installed)

Verify:

- Domain `corp.local` is visible
- OUs (Corp, Users, Servers) are present
- Users (alice, bob, itadmin) are present

#### Additional Validation

Run the following to confirm domain communication:

```cmd
echo %USERDOMAIN%
```

**Expected Result**

```text
CORP
```

Run the following to confirm domain connectivity:

```cmd
nltest /dsgetdc:corp.local
```

**Expected Result**

```text
DC: \\DC01
Address: \\192.168.56.10
```

### Step 10: Organize Server in Active Directory

Move SRV01 into the Servers OU:

```powershell
Get-ADComputer SRV01 | Move-ADObject -TargetPath "OU=Servers,OU=Corp,DC=corp,DC=local"
```

This step ensures the server is properly organized within Active Directory, allowing for future Group Policy application and administrative management.

### Step 11: Snapshot

A snapshot was taken after completing configuration:

```text
SRV01 - Domain Joined + RSAT Installed
```

This snapshot provides a stable restore point for future lab activities, ensuring a known-good baseline before introducing additional systems.

### Conclusion

SRV01 is now fully configured as a domain-joined management server. It provides a graphical interface for administering Active Directory, Group Policy, and DNS, serving as a central management point within the lab environment.

