# Domain Controller Setup (Windows Server 2025 Core)

## Overview
This document outlines the setup of Windows Server 2025 Domain Controller using Server Core inside of Oracle VirtualBox.

## Objective

- Configure Server Core for lab use
- Set static IP and DNS
- Install Active Directory Domain Services (AD DS)
- Promote the server to a Domain Controller
- Prepare network for clients to join the domain

## Environment

- VM Name: DC01
- OS: Windows Server 2025 Core
- Static IP: 192.168.56.10
- Subnet: 255.255.255.0
- DNS: 192.168.56.10

## Steps



> ⚠️ Installation issues? See [Installation Troubleshooting](../troubleshooting/dc-installation.md)

### Step 1: Initial SConfig Setup 

1. After installation, SConfig automatically launches.
2. **Rename Server**
   - Type `2` → Rename Computer
   - Enter `DC01`
   - Restart the server when prompted

### Step 2: Configure Network Settings

1. Type `8` → Network Settings

2. Select the network adapter connected to **Internal Network (LAB-NET)**

3. Assign a **static IP**:

   | Field         | Value                            |
   | ------------- | -------------------------------- |
   | IP Address    | 192.168.56.10                    |
   | Subnet Mask   | 255.255.255.0                    |
   | Gateway       | *(leave blank for isolated lab)* |
   | Preferred DNS | 192.168.56.10                    |
   | Alternate DNS | *(optional)*                     |

4. Confirm settings and exit



> ⚠️ Important: AD requires a static IP. NAT-assigned addresses (e.g., 10.0.2.15) will prevent domain functionality.



> ⚠️ Network issues? See [Network Troubleshooting](../troubleshooting/dc-network.md)

### Step 3: Enable Remote Desktop (Optional)

1. Type `7` → Remote Desktop
2. Press `E` to enable
3. Choose **1: Allow only clients running Remote Desktop with Network Level Authentication (NLA)**

### Step 4: Install Windows Updates

1. Type `6` → Download and Install Updates
2. Wait for installation and reboot if required

### Step 5: Install Active Directory Domain Services

1. Open PowerShell
2. Run the following to install and reboot if required

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

### Step 6: Promote to Domain Controller

1. Still in PowerShell, run

```powershell
Install-ADDSForest -DomainName "corp.local"
```

2. When prompted:
   - Enter **Directory Services Restore Mode (DSRM) password**
   - Confirm installation
   - Allow automatic reboot

### Step 7: Verify Domain Controller Functionality

1. Log in as `corp\\Administrator`
2. Check IP and DNS:

```powershell
ipconfig
nslookup dc01.corp.local
```

**Expected Results**

```text
Server: dc01.corp.local
Address: 192.168.56.10
Name: dc01.corp.local
Address: 192.168.56.10
```

3. Ensure that the DC is reachable from other lab VMs (later steps)

### Step 8: Initial Active Directory Configuration

After promoting the server to a Domain Controller, an initial Active Directory structure was created to simulate a basic enterprise environment.

### Create Organizational Units (OUs)

The following OU structure was created:

- Corp
  - Users
  - Computers
  - Servers

### Create Users

The following user accounts were created:

- alice
- bob
- itadmin

Each user was configured with:

- A UserPrincipalName (UPN) (e.g., `alice@corp.local`)
- A default password
- Forced password change at next logon

### Verify Active Directory Configuration

The following commands were used to verify user creation:

```powershell
Get-ADUser -Filter *
```

### Step 9: Domain Controller Validation

To ensure the Domain Controller was functioning correctly, the following checks were performed:

### DNS Resolution Tests

```powershell
nslookup corp.local
nslookup dc01.corp.local
```

**Expected Results**

```text
Server: dc01.corp.local
Address: 192.168.56.10

Name: corp.local
Address: 192.168.56.10
```

**Key Validation Points**

- DNS resolves correctly to the Domain Controller
- No stale or incorrect IP addresses present
- Active Directory users and OUs are accessible

### Step 10: Snapshot

A snapshot was taken after completing the configuration:

```text
DC - Clean (Post AD + DNS + Users)
```

This snapshot serves as a stable restore point before adding additional systems to the lab environment, ensuring a known-good baseline.

### Conclusion

This Domain Controller is now fully configured and validated, providing a stable foundation for additional systems in the lab environment.

