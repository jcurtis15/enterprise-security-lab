# Network Configuration and Connectivity

## Overview

This lab verifies network configuration and connectivity within the virtual lab environment. Basic networking commands were used to verify IP addressing, DNS resolution, and communication between domain-joined systems.

## Objective

- Verify IP configuration on domain-joined systems
- Confirm connectivity between lab machines
- Validate DNS name resolution
- Establish a baseline for network communication

## Environment

- Domain: corp.local
- Domain Controller: DC01 (192.168.56.10)
- Management Server: SRV01
- Client Machine: CLIENT01
- Network: LAB-NET (Internal Network)

## Steps Performed

### Step 1: Verify IP Configuration

On CLIENT01, Command Prompt was opened and the following command was executed:

```cmd
ipconfig /all
```

The following information was verified:

- IPv4 Address (192.168.56.x range)
- Subnet Mask (255.255.255.0)
- DNS Server (192.168.56.10)

> 💡 DNS configuration was verified using `ipconfig /all`, confirming the domain controller is being used for name resolution.

### Step 2: Test Connectivity to Domain Controller

From CLIENT01:

```cmd
ping 192.168.56.10
```

Result:

- Successful replies received from DC01

### Step 3: Test DNS Name Resolution

From CLIENT01:

```cmd
ping dc01
```

Result:

- Hostname successfully resolved to 192.168.56.10
- Replies received

> 💡 This confirms proper DNS functionality within the domain environment.

### Step 4: Test Connectivity Between Systems

From CLIENT01:

```cmd
ping srv01
```

From SRV01:

```cmd
ping client01
```

Result:

- Successful communication between systems

> 💡 This verifies all machines are on the same network and can communicate bidirectionally.

### Step 5: Analyze Network Path

From CLIENT01:

```cmd
tracert dc01
```

Result:

- Single hop observed

> 💡 This indicates direct communication within the same subnet.

### Step 6: Verify Domain Connectivity

From CLIENT01:

```cmd
echo %logonserver%
```

Result:

```text
\\DC01
```

> 💡 Confirms the client is authenticated against the domain controller.

### Conclusion

Network configuration and connectivity were successfully validated across all systems in the lab environment. IP addressing, DNS resolution, and host communication were confirmed to be functioning correctly.

This lab establishes a reliable network baseline required for domain operations, Group Policy functionality, logging, and future security testing scenarios.

This baseline ensures that all systems are properly connected and prepared for subsequent labs involving logging, monitoring, and security testing.