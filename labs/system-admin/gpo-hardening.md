# GPO Hardening

## Overview

This lab demonstrates the implementation of security-focused Group Policy Objects (GPOs) to harden a Windows workstation environment. Policies were configured to restrict user access to system tools and prevent the use of removable storage devices.

## Objective

- Restrict access to Control Panel and system settings
- Block the use of removable storage (USB devices)
- Apply both User and Computer configuration policies
- Validate enforcement from a domain-joined client system

## Environment

- Domain: corp.local
- Domain Controller: DC01 (Windows Server 2025)
- Management Server: SRV01 (Windows Server 2025, GUI)
- Client Machine: CLIENT01 (Windows 11)

## Steps Performed

### Step 1: Create and Link GPO

1. Open **Windows Tools** → **Group Policy Management**
2. Navigate through the domain structure:
   - Forest: corp.local
   - Domains → corp.local
   - Corp → Workstations
3. Right-click **Workstations**
4. Select:

```text
Create a GPO in this domain, and Link it here
```

5. Name the GPO:

```text
Workstation Hardening
```

### Step 2: Edit the GPO

1. In **Group Policy Management**, locate:
   - Corp → Workstations → Group Policy Objects
2. Right-click:

```text
Workstation Hardening
```

3. Select:

```text
Edit
```

This opens the **Group Policy Management Editor**.

### Step 3: Disable Control Panel (User Policy)

1. In the editor, navigate to:

```text
User Configuration → Policies → Administrative Templates → Control Panel
```

2. In the right pane, locate:

```text
Prohibit access to Control Panel and PC settings
```

3. Double-click the setting
4. Select:

```text
Enabled
```

5. Click **Apply** → **OK**

> 💡 This is a User Configuration policy and applies based on the location of user accounts within the OU.

### Step 4: Block USB Storage (Computer Policy)

1. In the editor, navigate to:

```text
Computer Configuration → Policies → Administrative Templates → System → Removable Storage Access
```

2. Locate:

```text
All Removable Storage classes: Deny all access
```

3. Double-click the setting
4. Select:

```text
Enabled
```

5. Click **Apply** → **OK**

> 💡 This is a Computer Configuration policy and applies based on the location of computer objects.

### Step 5: Configure Security Filtering

1. In **Group Policy Management**, select:

   - Corp → Workstations → Workstation Hardening

2. In the **Scope** tab, locate **Security Filtering**

3. Verify default entry:

   - **Authenticated Users** should already be present

4. Add the Employees group:

   - Click **Add**
   - Enter:

   ```text
   Employees
   ```

   - Click **Check Names**, then click **OK**

5. Configure permissions:

   - Click the **Delegation** tab
   - Click **Advanced**

   Ensure the following permissions:

   - **Authenticated Users:**
     - Read → Enabled
   - **Employees**:
     - Read → Enabled
     - Apply Group Policy → Enabled

> 💡 "Authenticated Users" allows the system to process the GPO, while "Employees" determines which users the policy applies to.

> 💡 This approach simplifies configuration while still allowing group-based targeting of policies.

### Step 6: Validate Policy Enforcement

On CLIENT01:

1. Log in as:

```text
corp\alice
```

2. Open **Run (Win + R)** and enter:

```text
gpupdate /force
```

3. Log out and log back in

> 💡 Command Prompt access is restricted by policy, so gpupdate was executed using the Run dialog.

> 💡 Computer Configuration policies (such as USB restrictions) may require a system restart if changes do not apply immediately.

#### Control Panel

- Attempt to open **Control Panel**
- Result: ❌ Access denied

#### USB Storage

- Insert a removable USB device (if available)
- Attempt to access the device
- Result: ❌ Access denied

> 💡 This lab demonstrates the combined use of User and Computer policies within a single GPO, a common approach in enterprise environments for centralized endpoint hardening and control.

## Conclusion

Security-focused Group Policy settings were successfully implemented to harden the workstation environment. Access to system configuration tools was restricted, and the use of removable storage devices was blocked.

This lab demonstrated the use of both User and Computer configuration policies within a single GPO, reinforcing how policy scope is determined by the location of user and computer objects in Active Directory.

Additionally, Security Filtering was used to ensure policies applied only to intended user groups, highlighting the importance of proper permission configuration when deploying Group Policy in enterprise environments.

These controls are commonly used in real-world environments to reduce attack surface, enforce organizational security policies, and protect systems from unauthorized access or data exfiltration.