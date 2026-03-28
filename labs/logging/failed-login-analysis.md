# Failed Login Simulation and Log Analysis

## Overview

This lab demonstrates how failed authentication attempts are recorded and analyzed within a Windows environment. Failed login attempts were intentionally generated and then reviewed using Event Viewer to understand how authentication failures appear in Security logs.

## Objective

- Generate failed login attempts
- Identify and analyze Event ID 4625
- Compare failed and successful authentication events
- Understand how suspicious login activity appears in logs

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01
- Client Machine: CLIENT01
- Network: LAB-NET (Internal Network)

## Steps Performed

### Step 1: Generate Failed Login Attempts

On CLIENT01:

1. Logged out of the current session
2. At the login screen, attempted to log in as:

```text
CORP\alice
```

3. Entered an incorrect password multiple times (3-5 attempts)

> 💡 This simulates failed authentication attempts such as password guessing or brute force behavior.

### Step 2: Log in Successfully

After generating failed attempts:

1. Logged in using valid credentials:

```text
CORP\alice
```

> 💡 This allows comparison between failed and successful authentication events.

### Step 3: Access Security Logs

Logged in as:

```text
CORP\itadmin
```

Then opened:

```text
Event Viewer → Windows Logs → Security
```

> 💡 Security logs are recorded at the system level and can be reviewed by authorized users, even if the account generating the activity does not have permission to view the logs.

### Step 4: Filter for Failed Login Events

1. Selected **Filter Current Log** from right pane
2. In the field labeled `<All Event IDs>`, entered:

```text
4625
```

Result:

- Multiple failed login events were identified for the `alice` account

### Step 5: Analyze Failed Login Events

Selected individual Event ID 4625 entries and reviewed:

- Account Name
- Failure Reason
- Logon Type
- Workstation Name
- Source Network Address

Observed:

- Repeated failed attempts for the same user
- Failure reason indicated incorrect password
- Logon Type showed interactive login attempts

### Step 6: Compare with Successful Login Events

Filtered for:

```text
4624
```

Compared successful login events with failed attempts:

- Same user account (`alice`)
- Different event outcomes (success vs failure)
- Similar logon type

> 💡 Comparing Event ID 4624 and 4625 helps distinguish between normal and suspicious authentication behavior.

### Step 7: Identify Suspicious Patterns

Observed:

- Multiple failed login attempts within a short timeframe
- Followed by a successful login

> ⚠️ This pattern may indicate password guessing or credential misuse and is commonly monitored in security operations.

> 💡 Patterns such as repeated failed logins followed by a successful login are commonly used in detection rules within SIEM platforms.

### Conclusion

Failed authentication attempts were successfully generated and analyzed using Event Viewer. Event ID 4625 was used to identify failed logins, while Event ID 4624 provided comparison with successful authentication events.

This lab demonstrates how authentication activity is recorded and how patterns in login behavior can be used to detect potential security threats such as brute force attacks or credential misuse.

Understanding how to generate, identify, and analyze these events is a foundational skill in security monitoring and incident detection.

This lab also builds a foundation for future SIEM integration, where such events can be aggregated and analyzed at scale for automated threat detection.

