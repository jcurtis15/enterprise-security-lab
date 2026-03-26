# Active Directory User & Group Management

This lab demonstrates the organization and management of user accounts and security groups within an Active Directory environment. The objective was to establish a structured identity management system and validate group-based access control across domain-joined systems.

## Objective

- Organize user accounts within Active Directory
- Create and manage security groups
- Assign users to appropriate groups
- Validate authentication and group membership from a domain-joined client

## Environment

- Domain: corp.local
- Domain Controller: DC01 (Windows Server 2025)
- Management Server: SRV01 (Windows Server 2025, GUI)
- Client Machine: CLIENT01 (Windows 11)

## Steps Performed

### Step 1: Review Existing OU Structure

An Organizational Unit (OU) structure was already in place:

- Corp
  - Users
  - Computers
  - Servers

### Step 2: Create Groups OU

1. Click **Start**  → **Windows Tools**
2. Open **Active Directory Users and Computers**
3. Navigate to: 
   - corp.local
4. Right-click  → **New  → Organizational Unit**
5. Name the OU:

```text
Groups
```

This OU is used to organize security groups.

### Step 3: Organize User Accounts

1. Navigate to:
   - corp.local  → Users (default container)
2. Identify existing users:
   - alice
   - bob
   - itadmin
3. Right-click each user  → **Move**
4. Move users to:

```text
Corp → Users
```

> ⚠️ The default "Users" container is not an OU and should not be used for structured environments.

> 💡 This behavior is due to security tokens being generated at logon, which includes group memberships.

### Step 4: Create Security Groups

1. Navigate to:
   - Corp → Groups
2. Right-click → **New** → **Group**
3. Create the following:

```text
Name: Employees
Type: Security
Scope: Global
```

```text
Name: IT
Type: Security
Scope: Global
```

### Step 5: Assign Users to Groups

1. Right-click group → **Properties**
2. Go to **Members**
3. Click **Add**

Assign:

- alice → Employees
- bob → Employees
- itadmin → IT

### Step 6: Validate Authentication

On CLIENT01, log in using:

```text
corp\alice
corp\bob
corp\itadmin
```

Run:

```cmd
whoami
```

### Step 7: Validate Group Membership

Run:

```cmd
whoami /groups
```

Confirm:

- alice and bob are members of **Employees**
- itadmin is a member of **IT**

> 💡 Group membership is applied at login. A logoff/logon may be required after changes.

### Conclusion

User accounts were successfully organized into the correct OU structure, and security groups were created to manage access control. Group membership was validated from a domain-joined client, confirming proper authentication and identity management within the environment.

This lab establishes a foundation for implementing permissions, Group Policy, and future security-focused scenarios.

