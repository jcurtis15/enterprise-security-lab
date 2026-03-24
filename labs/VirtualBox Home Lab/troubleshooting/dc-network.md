# Troubleshooting Domain Controller

This section documents common network configuration issues encountered during the Domain Controller setup in a VirtualBox-based lab environment.

## Network Configuration Issues

During the lab setup, several common issues can occur when configuring the static IP and DNS for the Domain Controller. Below is a summary of the problems encountered and their resolutions.

### Issue 1: APIPA Address (169.254.x.x) Appears

**Symptoms:**

After configuring a static IP in SConfig, running `ipconfig` shows an IP in the 169.254.x.x range.

**Cause:**

Windows assigns an Automatic Private IP Address (APIPA) when it cannot detect a network connection. This often happens if:

- The network adapter is misconfigured in VirtualBox
- The wrong adapter was configured in SConfig
- The adapter is not "Cable Connected" in VirtualBox

**Resolution**

1. Power off the VM.
2. Open **VirtualBox → Settings → Network → Adapter 1**.
   - Ensure it is **Attached to: Internal Network**
   - Name: `LAB-NET` (or your consistent lab network name)
   - Check **Cable Connected**
3. Start the VM and open SConfig → 8 → Network Settings
4. Select the correct adapter connected to LAB-NET
5. Set the static IP, subnet mask, and DNS (DC itself) again
6. Reboot the VM and verify with:

```powershell
ipconfig
nslookup dc01.corp.local
```

### Issue 2: Wrong Adapter Configured

**Symptoms**

After configuring a static IP, the server still shows an IP like 10.0.3.15 (NAT) or cannot ping other lab VMs.

**Cause**

- The static IP was applied to the NAT adapter instead of the Internal Network adapter
- The DC must use the adapter on the Internal Network for AD functionality

**Resolution**

1. Verify adapter names in PowerShell:

   ```powershell
   Get-NetAdapter
   ```

2. Make sure the adapter on **Internal Network (LAB-NET)** is the one being configured

3. Remove any old IPs on other adapters

```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet" -Confirm:$false
```

4. Apply static IP on the correct adapter:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.56.10 -PrefixLength 24
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.56.10
```

5. Verify with:

```powershell
ipconfig
nslookup dc01.corp.local
```

**Key Takeaways**

- Always configure the **Internal Network adapter** for static IP in lab environments.
- The **Preferred DNS** on the DC must point to itself.
- Use `Get-NetAdapter` to configure the correct adapter before setting IP addresses.
- APIPA address (169.254.x.x) indicates the server cannot reach a network adapter.
- Snapshots before network changes can save troubleshooting time.



