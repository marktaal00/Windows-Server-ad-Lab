# Print Server

## Goal

Deploy a shared printer via GPO to a specific department (HR), without access to physical printer hardware — using a software/virtual printer setup that still exercises the full real-world pipeline: role installation, sharing, permissions, and GPO-based automatic deployment.

## Setup

1. Installed the **Print and Document Services** role (Print Server role service) on the Domain Controller.
2. Added a printer using a **Standard TCP/IP Port** pointed at an unused IP on the lab subnet (since no physical printer was available) — sufficient for testing the sharing/deployment pipeline, since a printer object in Windows is fundamentally a driver + queue + share regardless of whether real hardware sits behind it.
3. Shared the printer and granted the `HR-Staff` security group **Print** permission via the printer's Security tab.
4. Deployed via GPO: **User Configuration → Preferences → Control Panel Settings → Printers → New → Shared Printer**, pointing to the printer's UNC path.

## Troubleshooting: virtual printer drivers can't be shared

**Symptom:** "Sharing is not supported by this type of printer" when attempting to share a printer created with the Microsoft Print to PDF or Microsoft XPS Document Writer driver.

**Root cause:** These are Microsoft-restricted virtual drivers, intentionally designed as local "print to file" only — Windows deliberately disables sharing for them.

**Fix:** Recreated the printer using a standard printer driver instead (first tried Microsoft IPP Class Driver, later Generic / Text Only) — any conventional driver supports sharing regardless of whether real hardware is attached.

## Troubleshooting: "Shared Printer" option missing from the GPO New menu

**Symptom:** Right-clicking under Printers in the GPO editor only showed "TCP/IP Printer" and "Local Printer" as options — no "Shared Printer."

**Root cause:** The "Shared Printer" deployment type only appears under **User Configuration** → Preferences → Printers. It does not appear under Computer Configuration, since connecting to a UNC-path printer share is inherently a per-user network connection concept, not a machine-level one.

**Fix:** Switched to editing the printer preference under User Configuration instead of Computer Configuration.

## Troubleshooting: client fails to connect — Error 0x00000490

**Symptom:** The printer worked and showed "Ready" status directly on the print server, but the client failed to connect with error `0x00000490` ("Element not found") — both manually via UNC path and via GPO deployment.

**Diagnosis:** Verified the printer's own queue status was healthy (ruling out a broken printer object), then tested manual UNC access (`\\Lab_Server\HR-Printer`) directly, which reproduced the same error outside of GPO entirely — isolating the problem to printer sharing/driver distribution, not the GPO configuration.

**Root cause:** Driver distribution mismatch. When a client connects to a shared printer, Windows must automatically download and install a matching driver from the server. The Microsoft IPP Class Driver — designed for genuinely IPP-discoverable network printers — did not distribute cleanly to the client in this fake-IP, no-real-hardware test scenario.

**Fix:** Removed the printer and recreated it using the **Generic / Text Only** driver, a simple, universally compatible driver well-suited to this kind of hardware-less test scenario. Cleared stale client-side printer objects and cached connections (`net use \\Lab_Server\HR-Printer /delete`) before retesting, to ensure a clean result.

**Verification:** Manually removed the printer from the client, ran `gpupdate /force`, logged off and back on, and confirmed the printer reappeared automatically under Printers & Scanners — without any manual reconnection — proving the GPO deployment path itself worked correctly once the underlying driver issue was resolved.

## Key takeaway

A GPO configuration can be entirely correct while the underlying resource it points to has its own independent failure mode (in this case, driver compatibility) — isolating whether a problem is "the GPO" or "the resource the GPO points to" (by testing the resource manually, outside of GPO, first) is a much faster way to root-cause this kind of issue than assuming it's a policy misconfiguration.
