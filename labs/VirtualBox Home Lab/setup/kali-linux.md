# Kali Linux Setup (KALI01)

## Overview

This document outlines the installation and configuration of a Kali Linux virtual machine (KALI01) within an Oracle VirtualBox lab environment. This system is configured as an attacker machine used for enumeration and security testing against the Active Directory domain.

## Objective

- Install Kali Linux in a virtual environment
- Configure network settings for lab connectivity
- Set static IP and DNS
- Validate connectivity to the domain environment
- Prepare the system for security testing and enumeration

## Environment

- VM Name: KALI01
- OS: Kali Linux (Debian-based)
- Static IP: 192.168.56.30
- Subnet: 255.255.255.0
- DNS: 192.168.56.10
- Network: LAB-NET (Internal Network)

## Steps

### Step 1: Virtual Machine Configuration

1. Created a new VM in VirtualBox:
   - Name: `KALI01`
   - Type: Linux
   - Version: Debian (64-bit)
2. Allocated Resources:
   - Memory: 4 GB
   - CPU: 2 cores
   - Disk: 50 GB (dynamically allocated)
3. Configured network:
   - Adapter 1:
     - Attached to: **Internal Network**
     - Name: `LAB-NET`
     - Cable Connected: Enabled

### Step 2: Install Operating System

1. Mounted Kali Linux ISO
2. Started VM and selected:

```text
Graphical Install
```

3. Configured installation settings:
   - Language: English
   - Location: United States
   - Keyboard: American English
4. Network configurations:
   - Selected: `Do not configure the network at this time`
5. Set system details:
   - Hostname: `kali`
   - Domain: *(left blank)*
6. Created user account:
   - Username: `kali`
   - Password: `(User-defined)`

### Step 3: Disk Partitioning

1. Selected:

```text
Guided - use entire disk
```

2. Chose:

```text
All files in one partition
```

3. Confirmed changes:
   - Selected **Yes** to write changes to disk



### Step 4: Complete Installation

1. Allowed installation to complete
2. Rebooted the system when prompted
3. Removed ISO if necessary to prevent reinstall loop

### Step 5: Initial Login

Logged into the system using:

```text
Username: kali
Password: (User-defined)
```

### Step 6: Configure Network Settings

1. Identified network interface:

```bash
ip a
```

2. Edited network configuration:

```bash
sudo nano /etc/network/interfaces
```

3. Configured static IP:

```bash
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
	address 192.168.56.30
	netmask 255.255.255.0
	dns-nameservers 192.168.56.10
```

4. Restarted networking services:

```bash
sudo systemctl restart networking
```

> 💡 The loopback interface (`lo`) is retained for local system communication and should not be removed.

### Step 7: Fix DNS Resolution

Kali initially attempted to use localhost for DNS, which caused resolution failures.

1. Edited DNS configuration:

```bash
sudo nano /etc/resolv.conf
```

2. Set DNS server:

```bash
nameserver 192.168.56.10
```

3. *(Optional)* Prevent overwriting:

```bash
sudo chattr +i /etc/resolv.conf
```

### Step 8: Verify Network Connectivity

Run the following:

```bash
ping 192.168.56.10
nslookup corp.local
```

**Expected Results**

```text
PING 192.168.56.10 successful

Server: 192.168.56.10
Name: corp.local
Address: 192.168.56.10
```

### Step 9: Validate Domain Visibility

```bash
nslookup dc01.corp.local
ping dc01.corp.local
```

**Expected Results**

```text
Name: dc01.corp.local
Address: 192.168.56.10
```

### Step 10: Snapshot

A snapshot was taken after completing the configuration:

```text
KALI01 - Network Configured
```

This snapshot provides a stable baseline before beginning security testing.

### Conclusion

KALI01 is now fully configured as an attacker machine within the lab environment. It has network connectivity to the domain and can resolve Active Directory resources. This system will be used for enumeration, testing, and cybersecurity lab exercises against the domain infrastructure.

