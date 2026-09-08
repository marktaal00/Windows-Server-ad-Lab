# Group Policy Deployment

## Goal

Practice creating, linking, scoping, and troubleshooting GPOs — including the more advanced concepts of precedence, Enforce, and Block Inheritance.

## GPOs built in this lab

- **Test-Policy** — Desktop Wallpaper (User Configuration), used as the first end-to-end GPO test
- **Restrict-CMD-PowerShell** — blocks Command Prompt / PowerShell access (User Configuration), scoped to a non-IT department
- **HR-Drive-Map** — auto-maps a network drive to HR department users (User Configuration Preferences), with Item-Level Targeting scoped to the `HR-Staff` security group
- **HR-Printer-Deploy** — deploys a shared printer automatically (User Configuration Preferences)
- **Default Domain Policy edits** — Password Policy and Account Lockout Policy (Computer Configuration, domain-wide — password policy specifically can only be set at the domain level, not on individual OUs)
- **IT-Password-Override** — a deliberately conflicting password policy GPO, created specifically to test and demonstrate GPO precedence, Enforce, and Block Inheritance behavior

## Troubleshooting: GPO not applying at all

**Symptom:** A wallpaper GPO linked to the `Workstations` OU did not apply, despite the client being correctly domain-joined and the computer object sitting in that OU.

**Diagnosis:**
```
gpresult /r
```
showed `N/A` under Applied Group Policy Objects for the user session.

**Root cause:** Desktop Wallpaper is a **User Configuration** setting. User Configuration policies apply based on where the *user account* sits in AD — not where the *computer* sits. The GPO was linked to `Workstations` (where the computer object lives), but the user account lived in a completely separate OU (`IT Department`). Since the two objects were in different OUs, the GPO never reached this user.

**Fix:** Linked the same GPO to the OU containing the user account instead (or alternatively, moved the setting to Computer Configuration if the intent was for it to apply regardless of who logs in). Confirmed the fix by re-running `gpresult /r` and seeing the GPO listed under Applied Group Policy Objects.

**Key lesson:** User Configuration settings follow the user's OU. Computer Configuration settings follow the computer's OU. These are frequently in different OUs by design (users are organized by department; computers are often organized by location/type), which is the most common reason a GPO "isn't working" despite being correctly created and linked.

## Troubleshooting: wallpaper applied but displayed as black

**Root cause:** The wallpaper path in the GPO pointed to a local server path (`C:\...`) that only existed on the DC, not on the client. GPOs deploy settings, not files — a local path on the server means nothing to the client machine.

**Fix:** Created a shared folder on the server and used a UNC network path (`\\Lab_Server\WallpaperShare\image.jpg`) instead of a local path, so the client could actually retrieve the file.

## GPO Precedence, Enforce, and Block Inheritance

Practiced the default precedence order (**LSDOU**: Local → Site → Domain → OU, with the closest/last-applied setting normally winning on conflict) by deliberately creating a conflicting password length policy at both the Domain level and an OU level:

1. **Default behavior:** the OU-level GPO (closer to the user) won over the Domain-level policy, as expected.
2. **After enabling "Enforced"** on the Domain-level GPO: the Domain policy overrode the OU-level GPO, despite normally losing precedence — demonstrating that Enforce deliberately breaks the default order.
3. **After also enabling "Block Inheritance"** on the OU: the Enforced Domain policy *still* applied. This confirmed the key interaction: **an Enforced GPO always overrides Block Inheritance** — a common interview question about GPO precedence.

## Key takeaway

Two lessons from this section came up repeatedly throughout the rest of the lab: (1) always check whether a setting is User or Computer Configuration and confirm the *relevant* object's OU, and (2) `gpresult /r` is the fastest way to confirm whether a GPO applied at all before troubleshooting the setting itself.
