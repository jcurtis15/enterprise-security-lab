# Ubuntu Server Setup (UBU01)

## Overview

This document outlines the installation and configuration of an Ubuntu Server virtual machine (UBU01) within an Oracle VirtualBox lab environment. This system is configured as a Linux-based server to simulate a mixed operating system environment and provide additional attack surface for security testing.

## Objective

- Install Ubuntu Server in a virtual environment
- Configure static IP and DNS
- Ensure connectivity to the domain environment
- Prepare the system for future security testing scenarios

## Environment

- VM Name: UBU01
- OS: Ubuntu Server (LTS)
- Static IP: 192.168.56.40
- Subnet: 255.255.255.0
- DNS: 192.168.56.10
- Network: LAB-NET (Internal Network)

## Steps

### Step 1: Virtual Machine Configuration

1. Created a new VM in VirtualBox:
   - Name: `UBU01`
   - Type: Linux
   - Version: Ubuntu (64-bit)
2. Allocated resources:
   - Memory: 4 GB
   - CPU: 2 cores
   - Disk: 40 GB (dynamically allocated)
3. Configured network:
   - Adapter 1:
     - Attached to: **Internal Network**
     - Name: `LAB-NET`
     - Cable Connected: Enabled

### Step 2: Install Operating System

1. Mounted Ubuntu Server ISO
2. Started VM and selected:

```text
Try or Install Ubuntu Server
```

3. Configured installation settings:
   - Language: English
   - Keyboard: Default (US)
4. Network configuration:
   - DHCP attempt failed (expected in isolated lab)
   - Continued without configuring the network
5. Configured system identity:

```text
Name: Jeff
Server name: ubu01
Username: ubuntu
Password: (user-defined)
```

6. Storage configuration:
   - Selected: Use entire disk
7. SSH Setup:
   - ✅ Installed OpenSSH Server
8. Skipped additional package selections

### Step 3: Complete Installation

1. Allowed installation to complete
2. Rebooted system when prompted
3. Removed ISO to prevent reinstall loop

### Step 4: Initial Login

Logged into the system using:

```text
Username: ubuntu
Password: (user-defined)
```

### Step 5: Configure Network Settings

1. Identified network interface:

```bash
ip a
```

2. Edited Netplan configuration:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

3. Configured static IP:

```yaml
network:
	version: 2
	ethernets:
		enp0s3:
			dhcp4: no
			addresses:
				- 192.168.56.40/24
			nameservers:
				addresses:
					- 192.168.56.10
```

4. Applied configuration:

```bash
sudo netplan apply
```

### Step 6: Fix DNS Resolution

Ubuntu initially used a local DNS resolver (`127.0.0.53`), which caused domain resolution failures. 

1. Disabled systemd-resolved:

```bash
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

2. Reconfigured DNS:

```bash
sudo rm /etc/resolv.conf
sudo nano /etc/resolv.conf
```

3. Added:

```text
nameserver 192.168.56.10
```

> 💡 Ubuntu uses systemd-resolved (127.0.0.53) as a local DNS stub, which can interfere with custom DNS configurations in lab environments.

### Step 7: Verify Network Connectivity

Run the following:

```bash
ping 192.168.56.10
nslookup corp.local
```

**Expected Results**

```text
PING successful

Server: 192.168.56.10
Name: corp.local
Address: 192.168.56.10
```

### Step 8: Validate Domain Connectivity

```bash
ping dc01.corp.local
nslookup dc01.corp.local
```

**Expected Results**

```text
Name: dc01.corp.local
Address: 192.168.56.10
```

### Step 9: Snapshot

A snapshot was taken after completing configuration:

```text
UBU01 - Network Configured
```

### Conclusion

UBU01 is now fully configured as a Linux server within the lab environment. It has proper network connectivity and can resolve domain resources. This system will be used to simulate a Linux-based target for enumeration, credential attacks, and lateral movement scenarios within the network.