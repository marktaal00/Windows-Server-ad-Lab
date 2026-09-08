# Folder Redirection

## Goal

Redirect a user's Desktop and Documents folders to a central network location via GPO, so the data survives hardware failure, supports centralized backup, and follows the user across different domain-joined machines — while remaining completely transparent from the user's perspective.

## Setup

1. Created a shared folder on the server (`C:\Redirected`), shared it, and configured NTFS permissions.
2. Configured the GPO: **User Configuration → Policies → Windows Settings → Folder Redirection** → Desktop and Documents → **Basic — Redirect everyone's folder to the same location** → **Create a folder for each user under the root path** → root path `\\Lab_Server\Redirected`.
3. Linked the GPO to the HR OU.

## Behavior worth noting: Folder Redirection requires a full logoff/logon

Running `gpupdate /force` while already logged in produced a warning:

> "The Group Policy Client Side Extension Folder Redirection was unable to apply one or more settings because the changes must be processed before system startup or user logon... OK to log off? (Y/N)"

This is expected, standard behavior for this specific GPO type — not an error. Folder Redirection must run before a session's Explorer/desktop environment is already using the old folder locations, so it cannot apply mid-session. Confirming with **Y** triggers the required logoff, after which the setting applies on the next logon.

## Troubleshooting: redirection appeared to process, but files still saved locally

**Symptom:** After completing the required logoff/logon cycle (twice), new files created on the Desktop/in Documents were still saving to the local `C:\Users\<username>\` path rather than the network location, despite no visible error.

**Diagnosis:** Checked the redirected root folder (`C:\Redirected`) on the server and found no per-user subfolder had been created at all for the test user — indicating the redirection engine could not write to the root folder, rather than a path or targeting mistake.

**Root cause:** The root folder's NTFS permissions only listed the department security group (`HR-Staff`), not the built-in **Authenticated Users** group. Folder Redirection's underlying mechanism specifically expects a broad "any authenticated domain user" permission at the root level, because it needs to create a new per-user subfolder there for whichever user is being redirected — a narrow custom group alone did not satisfy this, even though the specific test user was in fact a member of that group.

**Fix:** Added **Authenticated Users** with **Modify** permission to both the NTFS Security tab and the Share permissions of `C:\Redirected`, in addition to (not instead of) the existing `HR-Staff` entry. Re-ran `gpupdate /force`, completed the logoff/logon cycle, and confirmed the per-user subfolder was created and files saved there correctly.

**Verification:** Checked the Desktop folder's Properties → Location tab on the client (confirmed it now pointed to `\\Lab_Server\Redirected\<username>\Desktop`), then created a test file and confirmed it appeared in the corresponding server-side folder.

## Key concept: built-in groups vs. custom groups — choosing the right scope

This issue clarified an important distinction:
- **Authenticated Users** is a built-in, automatically-managed group representing any successfully domain-authenticated account — used when a resource genuinely needs to accommodate *any* user (like a redirection root folder, which is a container, not the actual private data).
- **Custom security groups** (`HR-Staff`, etc.) represent a deliberately narrow, manually managed population — the correct choice when a resource should be restricted to a specific department (like the department shared folders in the Permissions section of this lab).

Actual data privacy in Folder Redirection is preserved not by restricting the root folder narrowly, but because Windows automatically applies exclusive per-subfolder NTFS permissions once each user's own subfolder is created — meaning the root folder's broad accessibility does not compromise individual users' privacy.

## Key takeaway

Not every "restrict access" instinct applies the same way to every resource — a shared department folder and a Folder Redirection root folder look superficially similar (both are shared folders with NTFS permissions) but require fundamentally different permission scopes, because one holds the actual sensitive data directly, while the other is an infrastructure container whose real security boundary is enforced one level deeper.
