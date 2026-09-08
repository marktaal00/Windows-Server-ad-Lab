# Domain Controller Setup & DNS

## Environment

- **Hypervisor:** VMware Workstation
- **Server OS:** Windows Server 2019 (evaluation ISO)
- **Client OS:** Windows 11 Pro (domain-joined)
- **Domain:** `lab.local`

## Steps

### 1. Installed Windows Server 2019
Installed from a Microsoft evaluation ISO in VMware. Initially attempted to install directly from a USB-hosted ISO, which caused a "Windows cannot access the installation sources" error mid-install. Resolved by copying the ISO to the local C: drive before mounting it in the VM — USB read reliability during large file operations was the root cause.

### 2. Configured static IP and hostname
- Renamed the server (e.g., `Lab_Server`) before promoting to Domain Controller, since renaming after promotion is more complex.
- Set a static IP within the VMware NAT subnet (confirmed the correct subnet/gateway via **Virtual Network Editor → NAT Settings** first, rather than assuming).
- Preferred DNS server on the DC itself was set to `127.0.0.1` (points to itself, since it also hosts the DNS role).

### 3. Installed AD DS and promoted to Domain Controller
- Added the **Active Directory Domain Services** role via Server Manager.
- Promoted the server, creating a **new forest** with root domain `lab.local`.
- Skipped the DNS delegation prompt during promotion (expected/safe to skip for a standalone forest root — delegation is only relevant when integrating with an existing parent DNS zone).
- DNS Server role was installed automatically as part of DC promotion.

### 4. Verified the promotion
- Confirmed the domain appeared in Active Directory Users and Computers.
- Confirmed AD DS and DNS showed no warnings in Server Manager.

## Troubleshooting: client couldn't join the domain

**Symptom:** Domain join failed with *"An Active Directory Domain Controller for the domain 'lab.local' could not be contacted."*

**Diagnosis:**
```
nslookup lab.local
```
returned no result — confirming the issue was DNS resolution, not basic connectivity (`ping` to the DC's IP worked fine).

**Root cause:** The client's network adapter had DNS set to automatic/DHCP-assigned, which pointed to VMware's own NAT DHCP service rather than the domain controller. Since a domain requires clients to resolve DNS through the DC (to locate the DC itself and process authentication), this caused the join to fail outright.

**Fix:** Manually set the client's **Preferred DNS server** to the DC's static IP. Since the client used a static IP configuration overall, DNS also had to be set manually — Windows does not auto-populate DNS when IPv4 is set to "Use the following IP address."

```
ipconfig /flushdns
nslookup lab.local     # confirmed resolution after the fix
```

Domain join succeeded immediately after this fix.

## Key takeaway

DNS is the foundation almost everything else in AD depends on — domain join, GPO processing, and authentication all fail in confusing, indirect ways if DNS isn't pointed correctly at the DC. When troubleshooting AD issues generally, checking DNS resolution (`nslookup`) early is one of the fastest ways to rule in or out a whole category of problems.
