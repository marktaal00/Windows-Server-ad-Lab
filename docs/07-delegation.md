# Delegation of Control

## Goal

Simulate the delegated-administration model used at my actual workplace — a government district office where local IT has delegated rights over certain day-to-day tasks (like password resets), but Group Policy management is centralized at a regional/central office and is entirely outside local IT's access. The goal was to build and test this exact permission boundary hands-on, rather than only understanding it in the abstract.

## Setup

1. Created a standard (non-admin) test user to represent a "district-level IT support" account, with no membership in Domain Admins or any other privileged group.
2. Used the **Delegation of Control Wizard** on a single OU (HR) to grant this account specific, narrow rights:
   - Reset user passwords and force password change at next logon
   - Read all user information
   - Create, delete, and manage user accounts
3. Deliberately granted **no** Group Policy-related permissions.

## Testing

**Test 1 — delegated task within scope:** Logged in as the delegated account and successfully reset a password for a user within the HR OU.

**Test 2 — delegated task outside scope:** Attempted the same password reset action against a user in a different OU (Finance), which had not been delegated. Access was denied, confirming delegation is scoped strictly to the specific OU granted — not domain-wide.

**Test 3 — attempting GPO management:** Attempted to create/edit a Group Policy Object as the delegated account. This failed, confirming the account had zero Group Policy rights, matching the real-world central-office/district-office model.

## Troubleshooting: could not log into the Domain Controller directly as the delegated account

**Symptom:** "Sign-in method is not allowed" when attempting to log into the DC's console session using the delegated test account.

**Root cause:** By default, regular domain users (including delegated accounts with specific AD permissions) do not have rights to log on locally/interactively to a Domain Controller — only Domain Admins and specific privileged groups do. This is intentional security behavior, not a delegation misconfiguration.

**Resolution:** Rather than loosening DC logon rights (which would be poor practice in any real environment), tested the delegated account's permissions via **"Run as different user"** on the ADUC shortcut from an already-logged-in admin session — mirroring how, in a real environment, a delegated admin would use RSAT tools from their own workstation to manage AD remotely, without ever needing to log directly into the DC itself.

## Verifying delegated permissions directly

In addition to functional testing (attempting actions and observing allow/deny), verified delegation by inspecting the target OU's Security tab directly:
1. Enabled **View → Advanced Features** in ADUC (hidden by default).
2. Opened the delegated OU's Properties → **Security** tab, and confirmed the delegated account was explicitly listed there with the specific granted permissions — while non-delegated OUs showed no corresponding entry.

## Key takeaway

Delegation of Control is genuinely the mechanism behind a very common real-world organizational pattern: central IT retains policy-level control (Group Policy, domain-wide security settings) while local/regional IT staff get narrowly scoped operational rights (password resets, account management) limited to their own office's OU. Testing both "does the delegated action succeed within scope" and "does it correctly fail outside scope / for undelegated actions" is necessary to actually confirm delegation is working as intended — confirming only the positive case would miss a scoping mistake.
