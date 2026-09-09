# Windows Server Active Directory & Enterprise Lab

A hands-on Windows Server 2019 environment built in VMware Workstation to simulate a multi-department enterprise network. The lab demonstrates Active Directory administration, DNS, Group Policy, file and printer services, delegated administration, remote access, permissions management, and PowerShell automation.

**Environment:** VMware Workstation, Windows Server 2019 (Domain Controller), Windows 10/11 (domain-joined client)

## Overview

This lab was built from scratch — DC promotion through to VPN remote access — with a focus on simulating a realistic department-based organization (IT, Finance, HR, Engineering), similar in structure to my actual workplace (a government district office with delegated, non-Domain-Admin access).

## Key Skills Demonstrated

- Active Directory Domain Services setup, DNS configuration, static IP networking
- OU design, security groups, and department-based user structure
- Group Policy Objects (GPO): creation, linking, precedence, Enforce, Block Inheritance
- Group Policy Preferences: drive mapping, printer deployment, folder redirection
- NTFS and Share permission design (least-privilege, department-based access control)
- Print server deployment and GPO-based printer distribution
- Delegation of Control (simulating a real delegated-admin, non-Domain-Admin environment)
- Remote access: RDP with group-based restrictions, VPN via RRAS
- PowerShell for Active Directory administration (user/group management, password resets, account unlocks)

## Notable Troubleshooting

Real issues encountered and independently diagnosed and resolved during this project:

**1. GPO not applying — User vs. Computer Configuration scope**
A wallpaper GPO (User Configuration) failed silently because it was linked to the OU containing the *computer* object, not the *user* object. Diagnosed via `gpresult /r`, which showed "N/A" under Applied GPOs. Fixed by understanding that User Configuration settings follow the user's OU, not the computer's.

**2. "Everyone: Full Control" overriding NTFS restrictions**
Department shared folders remained accessible to unauthorized users despite correct NTFS permissions, because the Share tab still had the default "Everyone: Full Control" entry — which is checked first and less restrictive. Fixed by removing "Everyone" from Share permissions and scoping both Share and NTFS permissions to the correct security group.

**3. Printer driver mismatch — Error 0x00000490**
A GPO-deployed shared printer failed to connect from the client with error 0x00000490, despite working locally on the print server. Root cause was a driver distribution incompatibility with the Microsoft IPP Class Driver in a no-hardware test scenario. Resolved by switching to the Generic / Text Only driver, which distributes reliably to clients.

**4. Folder Redirection silently failing**
Desktop/Documents redirection appeared to process successfully (per `gpupdate` output) but files continued saving locally. Root cause: the redirection root folder only had a custom security group (department-specific) on its NTFS permissions, not the broader "Authenticated Users" group required for the redirection engine to create per-user subfolders. Fixed by adding Authenticated Users with Modify permission at the root, while relying on Windows' automatic per-subfolder exclusivity for actual data privacy.

**5. VPN (RRAS) connection failure**
VPN clients received "remote server is not responding" despite RRAS running and network connectivity confirmed via ping. Root cause: Windows Firewall inbound rules for RRAS/PPTP were not enabled by default after role installation. Fixed by enabling the relevant inbound firewall rules on the VPN server.

## Documentation

Detailed write-ups with screenshots for each stage are in [`/docs`](./docs):

1. [Domain Controller Setup & DNS](./docs/01-dc-setup.md)
2. [OUs, Security Groups & Users](./docs/02-ou-groups-users.md)
3. [Group Policy Deployment](./docs/03-gpo-deployment.md)
4. [Permissions & Shared Folders](./docs/04-permissions-shares.md)
5. [Print Server](./docs/05-print-server.md)
6. [Folder Redirection](./docs/06-folder-redirection.md)
7. [Delegation of Control](./docs/07-delegation.md)
8. [RDP & VPN Remote Access](./docs/08-rdp-vpn.md)
9. [PowerShell for AD](./docs/09-powershell.md)

## Background

Built to close specific gaps identified during a job interview, and to simulate delegated-administrator scenarios similar to my current role at a government IT office (Job Order IT Support Specialist), where GPO management sits with a central office and local IT operates under delegated permissions.
