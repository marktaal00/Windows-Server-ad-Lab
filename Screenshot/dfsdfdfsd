# PowerShell for Active Directory

## Goal

Practice the PowerShell equivalents of tasks already performed manually through ADUC across this lab — not as a replacement for GUI administration, but to understand when scripting is the more appropriate tool (bulk operations, consistency, automation, auditing/reporting), and to build baseline command familiarity for interviews.

## Commands practiced

**Import the AD module** (available by default on a DC with AD DS installed):
```powershell
Import-Module ActiveDirectory
```

**Create a user:**
```powershell
New-ADUser -Name "Test User" -GivenName "Test" -Surname "User" `
  -SamAccountName "tuser" -UserPrincipalName "tuser@lab.local" `
  -Path "OU=IT department,DC=lab,DC=local" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
  -Enabled $true
```

**Query users:**
```powershell
Get-ADUser -Filter * -SearchBase "OU=HR,DC=lab,DC=local"
Get-ADUser -Identity "lascuna" -Properties *
```

**Create a security group:**
```powershell
New-ADGroup -Name "Marketing-Staff" -GroupScope Global -GroupCategory Security `
  -Path "OU=IT department,DC=lab,DC=local"
```

**Manage group membership:**
```powershell
Add-ADGroupMember -Identity "HR-Staff" -Members "tuser"
Get-ADGroupMember -Identity "HR-Staff"
```

**Reset a password / unlock an account** (PowerShell equivalents of the delegated-admin tasks practiced manually in the Delegation of Control section):
```powershell
Set-ADAccountPassword -Identity "tuser" -Reset `
  -NewPassword (ConvertTo-SecureString "NewP@ss123!" -AsPlainText -Force)
Unlock-ADAccount -Identity "tuser"
```

**Move an object between OUs** (PowerShell equivalent of manually dragging a computer object into the correct OU):
```powershell
Move-ADObject -Identity "CN=Computer-01,CN=Computers,DC=lab,DC=local" `
  -TargetPath "OU=Workstations,DC=lab,DC=local"
```

## Troubleshooting: parameter not found error

**Symptom:** `New-ADUser` failed with a "parameter not found" error.

**Root cause:** A typo in a parameter name (`-Surename` instead of the correct `-Surname`). PowerShell parameter names must match exactly — an unrecognized parameter name produces this specific error rather than a more descriptive one.

**Fix:** Corrected the parameter spelling and re-ran the command successfully.

## When PowerShell is the better tool than the GUI

Reflecting on this after completing the lab primarily via ADUC:

- **Bulk operations** — creating/modifying many accounts at once (e.g., from a CSV of new hires) is dramatically faster and less error-prone via script than repeating a GUI wizard dozens of times.
- **Consistency** — a script applies identical logic every time; manual GUI repetition introduces a higher chance of human error across repeated actions.
- **Automation** — scripts can be scheduled to run unattended (e.g., disabling stale accounts, auto-unlocking known-safe recurring lockouts); GUI-driven tasks cannot.
- **Reporting/auditing** — tasks like "list all users who haven't logged in for 90 days" are one line in PowerShell and impractical through GUI browsing alone.
- **Environments without a GUI** — some real production servers (e.g., Server Core installs) do not have ADUC available at all, making PowerShell the only management option.

For genuinely one-off, individual changes, ADUC remains faster and perfectly appropriate — the two tools are complementary, not competing.

## Key takeaway

Having built a strong conceptual understanding of AD objects and permissions through the GUI first made the PowerShell cmdlets straightforward to pick up, since each command (`New-ADUser`, `Add-ADGroupMember`, `Move-ADObject`) maps directly onto an action already performed manually earlier in this lab.
