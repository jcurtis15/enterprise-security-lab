# Kali Linux Brute Force Attack Simulation (RDP)

## Overview

This lab demonstrates a real-world brute force attack simulation using Kali Linux against a Windows client system. The attack was performed using Hydra to target Remote Desktop Protocol (RDP) authentication on CLIENT01.

The resulting authentication activity was ingested into Splunk, where failed and successful login events were analyzed, correlated, and detected using previously developed detection logic and dashboards.

This lab builds on prior work by transitioning from simulated login activity to realistic attacker-generated traffic.

## Objective

- Simulate a brute force attack from Kali Linux
- Target RDP authentication on CLIENT01
- Generate real failed and successful login events
- Ingest attack logs into Splunk
- Validate detection logic against real attack data
- Identify attacker source IP within Splunk

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01 (Splunk Enterprise)
- Client Machine: CLIENT01 (Windows 11, domain-joined)
- Attacker Machine: Kali Linux
- Network: LAB-NET (Internal) + NAT (Internet access)

## Steps Performed

The following steps outline the full attack simulation process, from environment preparation to detection validation.

### Step 1: Configure Kali Networking

Kali Linux was configured with two network adapters:

- Adapter 1: Internal Network (`LAB-NET`)
- Adapter 2: NAT (Internet access)

Verified IP configuration:

```bash
ip a
```

Confirmed:

- `192.168.56.30` (LAB-NET)
- `10.0.3.X` (NAT)

Resolved routing and DNS issues to enable internet connectivity for package installation.

### Step 2: Install RDP Client

On Kali:

```bash
sudo apt update
sudo apt install freerdp-x11 -y
```

### Step 3: Enabled RDP on CLIENT01

On CLIENT01:

- Enabled Remote Desktop
- Disabled Network Level Authentication (NLA) for compatibility
- Allowed firewall rule:

```powershell
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

Added user access:

```powershell
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "CORP\bob"
```

### Step 4: Validate RDP Connectivity

From Kali:

```bash
xfreerdp /v:192.168.56.20 /u:'CORP\bob' /p:'Password1234!' /sec:rdp /cert:ignore
```

Confirmed successful RDP login to CLIENT01

### Step 5: Prepare Attack Wordlists

Created:

`users.txt`

Enter:

```text
alice
bob
```

Created:

`passwords.txt`

Entered:

```text
WrongPass1
WrongPass2
Password1234!
```

### Step 6: Execute Brute Force Attack

```bash
hydra -t 1 -W 1 -L users.txt -P passwords.txt rdp://192.168.56.20
```

Observed:

- Multiple failed login attempts
- Successful credential discovery for user `bob`

### Step 7: Verify Logs in Splunk

On SRV01:

```spl
index=main host=CLIENT01 (EventCode=4624 OR EventCode=4625)
| table _time, EventCode, Account_Name, Source_Network_Address
| sort - _time
```

Observed:

- EventCode 4625 → Failed logins
- EventCode 4624 → Successful login
- Source_Network_Address = `192.168.56.30`

> 💡 Unlike previous labs where activity originated from localhost (127.0.0.1), this attack shows a true external source (192.168.56.30), demonstrating accurate attacker attribution.

### Step 8: Validate Detection Logic

```spl
index=main host=CLIENT01 (EventCode=4625 OR EventCode=4624) 
| eval User=coalesce(TargetUserName, Account_Name) 
| bin _time span=5m 
| stats 
	count(eval(EventCode=4625)) as failed_attempts, 
	count(eval(EventCode=4624)) as successful_logins 
	by _time, User, Source_Network_Address 
| where failed_attempts >= 5 AND successful_logins >= 1
```

Detection successfully triggered for:

- User: bob
- Source: 192.168.56.30

### Step 9: Validate Dashboard

Observed dashboard updates:

- Failed login spikes during attack
- Successful login event upon credential discovery
- Top targeted user identified as `bob`
- Source panel displayed attacker IP
- Brute force detection panel triggered

### Key Concepts

- **Attack Simulation**: Generating realistic adversary behavior using Kali Linux
- **Source Attribution**: Identifying attacker IP from event logs
- **Authentication vs Authorization**: Differentiating login failures vs permission issues
- **Detection Validation**: Testing detection logic against real attack data
- **SIEM Workflow**: End-to-end pipeline from attack → logs → detection → visualization

### Challenges Encountered

- Kali networking and routing issues prevented internet access
- DNS resolution failures blocked package installation
- RDP connection failures due to NLA and Kerberos requirements
- User lacked RDP authorization permissions
- Hydra instability with RDP required tuning (`-t 1 -W 1`)

> ⚠️ Detailed troubleshooting steps can be found in [Kali RDP Brute Force Troubleshooting](../../troubleshooting/kali-rdp-brute-force-troubleshooting.md)

### Impact

- Delayed ability to simulate attack traffic
- Inability to validate detection logic until connectivity issues were resolved

### Conclusion

This lab demonstrates how a real brute force attack can be simulated and detected within a SIEM environment using Splunk.

By generating attack traffic from an external system and validating detection logic in Splunk, this lab provides a realistic demonstration of security monitoring and incident detection.

This represents a complete attack-to-detection workflow, including attacker simulation, log ingestion, correlation, and visualization.

### Next Steps

- Implement RDP-specific detection tuning (Logon_Type=10)
- Simulate additional attack types (SSH brute force, lateral movement)
- Enhance alerting based on detection logic