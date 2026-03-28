# GPO Scope and Security Filtering Issues

## Overview

This document outlines issues encountered during the implementation of Group Policy Objects (GPOs), specifically related to OU scope and security filtering. These issues prevented policies from applying correctly and required troubleshooting to resolve.

## Scope

This document focuses on common Group Policy issues related to OU placement, security filtering, and policy application behavior.

## Issue 1: GPO Not Applying to Users

### Problem

After creating and linking the GPO, policy settings (command prompt restriction and wallpaper) did not apply to users.

### Cause

The GPO was linked to the **Workstations OU**, but user accounts were located in a different Organizational Unit.

Since the policy was configured under **User Configuration**, it applies based on the location of user objects.

### Resolution

Moved user accounts into the same OU as the GPO:

```text
corp.local → Corp → Workstations
```

### Key Takeaway

> GPOs apply based on the location of user and computer objects within Organizational Units.

> User policies require the user account to be within the scope of the linked OU.

## Issue 2: GPO Applied to All Users (Including Admin Accounts)

### Problem

The GPO applied to all users in the Workstations OU, including administrative accounts (e.g., itadmin).

### Cause

The default **Security Filtering** setting includes:

```text
Authenticated Users
```

This causes the GPO to apply to all authenticated users.

### Resolution

Modified Security Filtering to restrict policy application:

1. Removed **Authenticated Users**
2. Added **Employees** group

### Key Takeaway

> Security Filtering determines which users or groups a GPO applies to, allowing for role-based access control.

## Issue 3: GPO Stopped Applying After Security Filtering Changes

### Problem

After modifying Security Filtering, the GPO stopped applying entirely.

- Wallpaper no longer applied
- Command prompt was accessible

### Cause

Removing **Authenticated Users** removed required **Read permissions**, preventing both user and computer accounts from processing the GPO.

### Resolution

Reconfigured Security Filtering and permissions:

1. Re-added **Authenticated Users** for required read access
2. Modified permissions under **Delegation** → **Advanced**:

- Authenticated Users:
  - Read → Enabled
  - Apply Group Policy → Disabled
-  Employees:
  - Read → Enabled
  - Apply Group Policy → Enabled

### Key Takeaway

> GPOs require Read permissions to be processed, even if they are not applied.

> Security Filtering controls application, while Read permission enables processing.

## Issue 4: Policy Changes Not Immediately Applied

### Problem

Changes made to the GPO did not immediately reflect on the client system.

### Cause

Group Policy updates require a refresh cycle and user logon to fully apply changes.

### Resolution

On CLIENT01:

```cmd
gpupdate /force
```

Then log out and log back in.

### Key Takeaway

> User-based policies are applied at logon and may require a session refresh to take effect.

## Summary

The following concepts were reinforced during troubleshooting:

- OU structure determines GPO scope
- Security Filtering controls who receives the policy
- Read permissions are required for GPO processing
- Policy changes require refresh and reauthentication

Proper configurations of these elements are essential for successful Group Policy implementation and troubleshooting in enterprise environments. 