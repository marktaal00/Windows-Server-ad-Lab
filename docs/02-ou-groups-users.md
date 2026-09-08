# OUs, Security Groups & Users

## Goal

Simulate a realistic department-based organization structure, similar to a real office environment (IT, Finance, HR, Engineering departments), to practice OU design, security group usage, and delegated permissions in a context close to my actual workplace.

## Structure built

```
lab.local
├── IT Department (OU)
│   ├── User (e.g., jsmith)
│   └── IT-Staff (Security Group)
├── Finance (OU)
│   ├── User
│   └── Finance-Staff (Security Group)
├── HR (OU)
│   ├── User
│   └── HR-Staff (Security Group)
├── Engineering Department (OU)
│   ├── User
│   └── Engineering-Staff (Security Group)
└── Workstations (OU)
    └── Domain-joined client computer objects
```

## Key concepts practiced

### OU vs. Group — a common early point of confusion
An **OU (Organizational Unit)** is a container — used to organize objects and to link Group Policy Objects. A **Group** is an object *inside* an OU — used to manage permissions and GPO targeting. A user can only exist in one OU, but can belong to many groups simultaneously — this is the main reason groups (not OU membership) are the standard tool for controlling access to resources.

### Security Group vs. Distribution Group
- **Security groups** control access — file/folder permissions, GPO security filtering, VPN access, etc. This is the default choice for any IT administration task.
- **Distribution groups** are email-only mailing lists and cannot be used for permissions or GPO filtering at all.

### Default computer object placement
By default, a newly domain-joined computer lands in the built-in **"Computers"** container — not a real OU, and GPOs cannot be linked to it. Computers need to be manually moved into a proper OU (or automatically redirected — see below) for Computer Configuration GPO settings to apply.

**Fix applied:** ran `redircmp` on the DC to permanently redirect all future computer joins directly into the `Workstations` OU, removing the need to manually move each new machine:
```
redircmp "OU=Workstations,DC=lab,DC=local"
```

## Key takeaway

Every user in this lab was deliberately added to a matching department security group, not just placed in the correct OU — because permissions, GPO Security Filtering, and Group Policy Preferences (drive maps, printers) are all built around group membership, not OU location. Understanding this distinction — and that OU membership and group membership serve two different purposes — was foundational to everything else in this lab.
