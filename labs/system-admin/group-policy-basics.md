# Group Policy Basics

## Overview

This lab demonstrates the creation and application of Group Policy Objects (GPOs) within an Active Directory environment. Policies were configured to restrict user access and enforce desktop settings, then validated from a domain-joined client system.

## Objective

- Create and link a Group Policy Object (GPO)
- Apply user-based policy settings
- Control policy scope using Organizational Units (OUs)
- Validate policy enforcement from a client system

## Environment

- Domain: corp.local
- Domain Controller: DC01 (Windows Server 2025)
- Management Server: SRV01 (Windows Server 2025, GUI)
- Client Machine: CLIENT01 (Windows 11)

## Notes

This lab includes references to troubleshooting scenarios encountered during implementation. Detailed explanations and resolutions are documented separately.

## Steps Performed

### Step 1: Create Workstations OU

1. Open **Windows Tools** → **Active Directory Users and Computers**
2. Navigate to:
   - corp.local → Corp
3. Right-click → **New** → **Organizational Unit**
4. Name the OU:

```text
Workstations
```

### Step 2: Move Objects into OU

Moved the following into the Workstations OU:

- CLIENT01 (computer)
- alice (user)
- bob (user)
- itadmin (user)

### Step 3: Create and Link GPO

1. Open **Windows Tools** → **Group Policy Management**
2. Navigate to:
   - corp.local → Corp → Workstations
3. Right-click → **Create a GPO in this domain, and Link it here**
4. Name the GPO:

```text
Workstation Restrictions
```

### Step 4: Configure GPO Settings

**Disable Command Prompt**

```text
User Configuration → Policies → Administrative Templates → System
```

Enable:

```text
Prevent access to the command prompt
```

**Set Desktop Wallpaper**

```text
User Configuration → Policies → Administrative Templates → Desktop → Desktop
```

Enable:

```text
Desktop Wallpaper
```

Set:

```text
C:\Windows\Web\Wallpaper\Windows\img0.jpg
```

> 💡 These settings are applied under User Configuration and affect users based on their OU location.

### Step 5: Apply and Validate Policy

On CLIENT01:

1. Log in as:

```text
corp\alice
```

2. Run:

```cmd
gpupdate /force
```

3. Log out and log back in

Repeat this process for bob and itadmin to ensure consistent policy behavior across user accounts.

> ⚠️ Initial policy application issues were encountered due to OU scope misalignment.

> See [Troubleshooting: GPO Scope and Filtering](../troubleshooting/gpo-scope-and-filtering.md)

### Step 6: Verify Results

**alice (Employees group)**

- Command Prompt → ❌ Blocked
- Desktop Wallpaper → ✅ Applied

**itadmin (IT group)**

- Command Prompt → ✅ Accessible
- Desktop Wallpaper → ❌ Not applied

> ⚠️ Additional issues were encountered when configuring Security Filtering, which temporarily prevented the GPO from applying.

> See [Troubleshooting: GPO Scope and Filtering](../troubleshooting/gpo-scope-and-filtering.md)

### Conclusion

Group Policy Objects were successfully created and applied within the domain environment. Policy enforcement was validated from a client system, demonstrating centralized control of user behavior using Active Directory.