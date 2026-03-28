# Log Correlation and Event Timeline Analysis

## Overview

This lab demonstrates how multiple authentication events can be correlated to identify patterns of user activity within a Windows environment. Rather than analyzing individual events in isolation, related events were examined together to build a timeline and better understand user behavior.

## Objective

- Correlate multiple authentication events
- Build a timeline of login activity
- Identify relationships between failed and successful logins
- Understand how event patterns can indicate suspicious behavior

## Environment

- Domain: corp.local
- Domain Controller: DC01
- Management Server: SRV01
- Client Machine: CLIENT01
- Network: LAB-NET (Internal Network)

## Steps Performed

### Step 1: Generate Authentication Activity

On CLIENT01:

1. Logged out of the current session
2. Attempted to log in as:

```text
CORP\alice
```

3. Entered an incorrect password multiple times (4-6 attempts)
4. Successfully logged in using valid credentials

> 💡 This created a sequence of failed and successful authentication events for analysis.

### Step 2: Access Security Logs

Logged in as: 

`CORP\itadmin`

Then opened:

```text
Event Viewer → Windows Logs → Security
```

### Step 3: Filter Relevant Events

Used **Filter Current Log** and entered:

```text
4624,4625
```

Result:

- Both failed (4625) and successful (4624) login events were displayed

### Step 4: Sort Events by Time

Sorted events using the **Date and Time** column to view activity in chronological order.

> 💡 Sorting events chronologically allows for the reconstruction of user activity over time.

### Step 5: Build Event Timeline

Identified a sequence of related events:

```text
4625 - Failed login (alice)
4625 - Failed login (alice)
4625 - Failed login (alice)
4624 - Successful login (alice)
```

### Step 6: Correlate Event Details

Compared the following fields across events:

- Account Name
- Logon Type
- Workstation Name
- Timestamp

Observed:

- All events involved the same user (`alice`)
- Events occurred within a short timeframe
- Logon type remained consistent
- All activity originated from the same system

> 💡 Matching event attributes confirms that the events are part of the same activity sequence.

### Step 7: Analyze Behavior Pattern

The following pattern was observed:

- Multiple failed login attempts
- Followed by a successful login

> ⚠️ This pattern may indicate password guessing, user error, or potential credential misuse and should be investigated in real-world environments.

> 💡 Correlating multiple events provides better visibility into user behavior than analyzing individual events in isolation.

### Conclusion

Multiple authentication events were successfully correlated to build a timeline of user activity. By analyzing Event IDs 4625 and 4624 together, a clear sequence of failed and successful login attempts was identified.

This lab demonstrates the importance of event correlation in detecting patterns that may indicate suspicious behavior. Rather than relying on single events, analyzing event sequences provides deeper insight into system activity and potential security threats.

This approach forms the foundation for detection engineering and SIEM-based monitoring, where large volumes of events are analyzed collectively to identify malicious behavior.