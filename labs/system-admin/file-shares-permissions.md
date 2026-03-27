# File Shares and Permissions

## Overview

This lab demonstrates the creation and configuration of shared folders within a Windows Server environment, using Active Directory security groups to control access. Both NTFS and share permissions were configured and validated from a domain-joined client system.

## Objective

- Create shared folders on a server
- Configure NTFS and share permissions
- Apply group-based access control using Active Directory
- Validate access from a domain-joined client

## Environment

- Domain: corp.local
- Domain Controller: DC01 (Windows Server 2025)
- Management Server: SRV01 (Windows Server 2025, GUI)
- Client Machine: CLIENT01 (Windows 11)

## Steps Performed

### Step 1: Create Shared Folders

On SRV01, the following directories were created:

```text
C:\Shares\Public
C:\Shares\IT
```

### Step 2: Configure NTFS Permissions

1. Right-click folder → **Properties** → **Security**
2. Click **Edit** → **Add**
3. Enter group name → **Check Names** → **OK**

Permissions were configured as follows:

**Public Folder**

- Employees → Read & Execute
- IT → Modify

**IT Folder**

- IT → Modify
- Employees → Removed (no access)

> ⚠️ After modifying permissions, the **Apply** button must be selected to save changes.
>
> 💡 The "Check Names" function verifies the group exists in Active Directory and resolves it to the correct object.

### Step 3: Configure Share Permissions

1. Right-click folder → **Properties** → **Sharing**
2. Click **Advanced Sharing**
3. Check **Share this folder**
4. Click **Permissions**

**Public Folder**

- Everyone → Read

**IT Share**

- IT → Full Control
- Removed Everyone

> ⚠️ Share permissions cannot be configured until "Share this folder" is enabled.
>
> 💡 Share permissions control network access, while NTFS permissions control file system access.

### Step 4: Validate Initial Access

On CLIENT01, access was tested using:

```text
Win + R → \\SRV01
```

> 💡 UNC paths (e.g., \\SRV01) are used to access network resources in Windows environments.

Observed results:

- Public share was accessible as expected
- IT share was restricted as expected
- An additional shared folder (`Shares`) was visible

### Step 5: Identify Misconfiguration

The parent directory (`C:\Shares`) was also shared, which allowed users to access restricted subfolders indirectly:

```text
\\SRV01\Shares
```

This created a permission bypass scenario, allowing unintended access to restricted resources through an alternate access path.

### Step 6: Resolve Parent Share Misconfiguration

To correct this issue:

1. On `SRV01`, right-click `C:\Shares` → **Properties**
2. Navigate to **Sharing** → **Advanced Sharing**
3. Uncheck **Share this folder**
4. Click **Apply** → **OK**

> ⚠️ See [Troubleshooting: File Share Permission Bypass](../troubleshooting/file-share-permission-bypass.md)

### Step 7: Validate Final Access

Access was re-tested from CLIENT01:

**alice (Employees group)**

- Access Public → ✅ Allowed (Read)
- Access IT → ❌ Denied

**itadmin (IT group)**

- Access Public → ✅ Allowed
- Access IT → ✅ Allowed (Modify)

### Conclusion

Shared folders were successfully configured using both NTFS and share permissions. Access control was enforced using Active Directory groups, and behavior was validated from a client system.

A misconfiguration involving a shared parent directory was identified and corrected, reinforcing the importance of proper share design and permission layering.

This lab demonstrates how access control is implemented in real-world environments and highlights the interaction between NTFS and share permissions.

This scenario highlights how improper share configuration can lead to unintended access, a common issue encountered in real-world environments.