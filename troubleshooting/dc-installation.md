# Troubleshooting Domain Controller Installation

This document outlines common installation issues encountered while setting up a Windows Server 2025 Domain Controller in a VirtualBox-based lab environment.

## Issue 1: Installer Requests Disk Unexpectedly

**Symptoms**

After starting the VM and launching the Windows Server installer, the system prompts for a disk or fails to detect installation media.

**Cause**

The VM boot order is not correctly configured, and the system is not prioritizing the mounted ISO.

**Resolution**

1. Power off the VM.
2. Open **VirtualBox → Settings → System → Motherboard**.
3. Under **Boot Order**, move **Optical** to the top.
4. Start the VM again and proceed with installation.

## Issue 2: Cannot Use Disk Partition During Installation

**Symptoms**

Windows Server setup does not allow selecting the available disk partition for installation.

**Cause**

The virtual disk may contain leftover or improperly formatted partition data from a previous attempt.

**Resolution**

1. At the disk selection screen, press **Shift + F10** to open Command Prompt.
2. Launch DiskPart:

```cmd
diskpart
list disk
select disk 0
clean
exit
```

3. Close Command Prompt.
4. Click **Refresh** in the installer.
5. Select the disk and continue the installation.



## Key Takeaways

- Ensure the **ISO is prioritized in boot order** to avoid installation media issues.
- Use **DiskPart** `clean` to resolve partition-related errors.
- Virtual disks can retain previous configurations that interfere with installation.
- Troubleshooting early setup issues saves time later in the lab build.