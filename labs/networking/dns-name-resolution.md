# DNS and Name Resolution

## Overview

This lab demonstrates how DNS functions within an Active Directory environment and its critical role in domain operations. Name resolution was tested under normal conditions, then intentionally misconfigured to observe the impact on domain functionality and troubleshoot the issue.

## Objective

- Verify DNS configuration on a domain-joined system
- Test hostname and FQDN resolution
- Simulate a DNS misconfiguration
- Observe and troubleshoot name resolution failures
- Restore proper DNS functionality

## Environment

- Domain: corp.local
- Domain Controller: DC01 (192.168.56.10)
- Management Server: SRV01
- Client Machine: CLIENT01
- Network: LAB-NET (Internal Network)

## Steps Performed

### Step 1: Verify DNS Configuration

On CLIENT01, Command Prompt was opened and the following command was executed:

```cmd
ipconfig /all
```

The DNS server was confirmed as:

```text
192.168.56.10
```

> 💡 This confirms the client is using the domain controller for DNS resolution.

### Step 2: Test DNS Name Resolution

From CLIENT01:

```cmd
nslookup dc01
```

Result:

- Hostname successfully resolved to 192.168.56.10

### Step 3: Test FQDN Resolution

From CLIENT01:

```cmd
ping dc01.corp.local
```

Result:

- Successful resolution and replies received

> 💡 This verifies proper DNS suffix configuration and domain-based name resolution.

### Step 4: Misconfigure DNS (Simulate Failure)

On CLIENT01:

1. Open **Control Panel**
2. Navigate to:
   - **Network and Internet** → **Network and Sharing Center**
3. Click **Change adapter settings**
4. Right-click **Ethernet** → **Properties**
5. Select:

```text
Internet Protocol Version 4 (IPv4)
```

6. Click **Properties**
7. Under **Use the following DNS server addresses**, change to:

```text
8.8.8.8
```

8. Click **OK**, then **Close**

> ⚠️ This removes dependency on the internal domain DNS server.

### Step 5: Observe Name Resolution Behavior

From CLIENT01:

```cmd
nslookup dc01
```

Result:

- DNS resolution failed when using nslookup

Now run:

```cmd
ping dc01
```

Result:

- Still returned successful replies

Now run:

```cmd
echo %logonserver%
```

Result:

```text
\\DC01
```

### Step 6: Analyze Results

> ⚠️ Although DNS was misconfigured, `ping dc01` continued to succeed due to cached entries or fallback name resolution methods such as NetBIOS or LLMNR.

> 💡 The `nslookup` command relies strictly on DNS and accurately reflects DNS functionality, making it the preferred tool for testing name resolution.

### Step 7: Restore DNS Configuration

To correct the issue:

1. Return to:
   - **Control Panel** → **Network and Sharing Center** → **Change adapter settings**
2. Right-click **Ethernet** → **Properties**
3. Select **Internet Protocol Version 4 (IPv4)** → **Properties**
4. Set DNS server to:

```text
192.168.56.10
```

5. Click **OK**, then **Close**

### Step 8: Flush and Verify DNS

From CLIENT01:

```cmd
ipconfig /flushdns
```

Then test:

```cmd
nslookup dc01
ping dc01
```

Result:

- DNS resolution restored successfully
- Normal communication resumed

### Conclusion

DNS functionality within the Active Directory environment was successfully validated and tested under both normal and misconfigured conditions. Changing the DNS server to a public address disrupted proper domain name resolution, demonstrating the dependency of Active Directory on internal DNS.

Despite the misconfiguration, some commands such as `ping` continued to function due to cached entries and fallback resolution methods, highlighting the importance of using appropriate tools like `nslookup` for accurate DNS troubleshooting.

This lab reinforces the critical role of DNS in domain environments and the potential impact of misconfiguration on authentication, resource access, and overall network functionality.

This scenario highlights how DNS misconfigurations can create misleading symptoms, emphasizing the importance of using the correct tools when troubleshooting network and domain-related issues.