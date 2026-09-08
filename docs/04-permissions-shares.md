# Permissions & Shared Folders

## Goal

Build department-restricted shared folders (Finance, HR, IT, Engineering), where only the matching department's security group can access their own folder — and verify the restriction actually holds under testing, not just in theory.

## Structure

```
C:\Shares\
├── Finance   → Finance-Staff group only
├── HR        → HR-Staff group only
├── IT        → IT-Staff group only
└── Engineering → Engineering-Staff group only
```

## Permission model used

For each folder:
- **Share permissions** (Sharing tab → Advanced Sharing → Permissions): granted the matching `-Staff` security group **Full Control**
- **NTFS permissions** (Security tab): granted the same group **Modify** (not Full Control — Modify allows normal read/write/delete, without letting regular staff change permissions or take ownership)

## Troubleshooting: a Finance user could access the HR folder

**Symptom:** Despite NTFS permissions on the HR folder only listing `HR-Staff`, a Finance department test user was still able to browse into and access the HR shared folder.

**Diagnosis:** Checked the HR folder's **Sharing → Advanced Sharing → Permissions** tab (separate from the NTFS Security tab) and found the default **"Everyone: Full Control"** entry still present — left over from when the folder was first shared.

**Root cause:** When a resource is accessed over the network, Windows evaluates **both** Share permissions and NTFS permissions, and applies whichever is **more restrictive**. However, "Everyone: Full Control" on the Share tab is permissive enough that it doesn't restrict anything on its own — so if NTFS is expected to be the only real gatekeeper but "Everyone" is still granting broad share-level access, the *combination* still allows through anything NTFS doesn't explicitly block. In this case, the NTFS permissions had not yet been correctly scoped either, and "Everyone: Full Control" left on the Share tab was masking the misconfiguration until permissions were properly audited on both tabs.

**Fix:**
1. Removed **"Everyone"** from the Share permissions tab entirely.
2. Added the matching department security group to Share permissions with Full Control.
3. Confirmed the NTFS Security tab correctly listed only the matching group with Modify.
4. Used **Advanced → Effective Access** (Security tab) to verify, per-user, exactly what access a given test user would have — before manually testing via login, to catch misconfigurations early.

**Verification:** Logged in as the Finance test user and confirmed access to `\\Lab_Server\Shares\HR` was denied, while access to `\\Lab_Server\Shares\Finance` remained available. Repeated for each department.

## Key concepts

**Share permissions vs. NTFS permissions**
- Share permissions are the "front door" — they only apply when a resource is accessed *over the network*.
- NTFS permissions are the "actual lock on the room" — they apply always, whether accessed locally or remotely.
- Real-world best practice: set Share permissions loosely (e.g., grant the relevant group Full Control there) and do all genuine fine-grained access control at the NTFS level — this avoids having to keep two separate permission systems in sync, and reduces the exact class of bug encountered above.

**"Everyone" vs. "Authenticated Users"**
Learned (and later reinforced during Folder Redirection troubleshooting) that "Everyone" should generally be avoided in modern environments in favor of "Authenticated Users" — the latter guarantees only genuinely domain-authenticated accounts are covered, with no dependency on anonymous-access settings elsewhere in the environment. Leaving "Everyone" on a share, even if functionally similar in a well-configured environment, is generally considered a misconfiguration/finding in a real security review.

## Key takeaway

A resource can look correctly locked down on paper (NTFS Security tab lists only the intended group) while still being effectively open, because Share permissions are a separate, easy-to-forget gate. Auditing *both* tabs — and verifying with Effective Access rather than assuming — is the reliable way to avoid this exact class of misconfiguration.
