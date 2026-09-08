# RDP & VPN Remote Access

## Goal

Practice both common forms of Windows remote access: Remote Desktop (remoting into a specific machine's desktop) and VPN via RRAS (remote network-level access), including group-based access restriction for RDP.

## Remote Desktop (RDP)

### Setup
1. Enabled Remote Desktop on the target client machine (Settings → System → Remote Desktop).
2. Restricted access to a specific security group instead of leaving it at the default (local Administrators only): added the `IT-Staff` domain security group under **"Select users that can remotely access this PC."**
3. Connected from a separate machine using **Remote Desktop Connection (`mstsc`)**.

### Verification
- Confirmed a user who is a member of `IT-Staff` could successfully connect.
- Confirmed a user who is **not** a member of `IT-Staff` (a Finance department test user) was denied — verifying the group-based restriction actually took effect, not just that RDP itself worked.

### Local vs. domain account distinction
To connect using the machine's local Administrator account instead of a domain account, the local account had to be explicitly specified (`.\Administrator` or `ComputerName\Administrator`) — typing just `Administrator` alone defaults to attempting domain authentication first on a domain-joined machine. This is the same local-vs-domain ambiguity encountered earlier during initial domain join troubleshooting.

### NLA (Network Level Authentication)
Confirmed NLA was enabled (the default, recommended setting). NLA requires credential verification to complete **before** a full remote session is established, rather than after a session/login screen has already loaded — reducing exposure to attacks that target the pre-authentication session negotiation process, and reducing resource load from unauthenticated connection attempts.

## VPN via RRAS (Routing and Remote Access Service)

### Setup
1. Installed the **Remote Access** role with the **DirectAccess and VPN (RAS)** role service.
2. Used the traditional **"Configure and Enable Routing and Remote Access"** wizard (Custom configuration → VPN access) — deliberately avoided the more complex "Deploy VPN and DirectAccess" wizard, which is intended for advanced always-on scenarios and unnecessary for this test.
3. Configured a **Static address pool** for VPN clients under the server's IPv4 properties (a range outside the existing static IP assignments).
4. Set **Network Access Permission → Allow access** on the Dial-in tab for the test user account.
5. Configured a VPN connection on the client (Windows built-in VPN client), initially using PPTP.

### Troubleshooting: "remote server is not responding"

**Diagnosis process:**
1. Confirmed RRAS was actually running (green arrow in the console) — ruled out.
2. Confirmed the connection attempt originated from a VM on the same internal network as the server, not an unrelated external network — ruled out a network-topology mismatch.
3. Confirmed basic connectivity (`ping` to the server's IP) succeeded — ruled out DNS/routing issues.
4. Checked **Windows Firewall inbound rules** on the server and found the RRAS/PPTP-related inbound rules were not enabled.

**Fix:** Enabled the relevant inbound firewall rules (Routing and Remote Access PPTP-In / related rules) on the server. Retested and the VPN connected successfully.

### VPN protocol notes (PPTP, L2TP/IPsec, SSTP, IKEv2)
Explored the practical differences between the built-in Windows VPN protocol options:
- **PPTP** — simplest to configure, but considered obsolete/insecure for real production use; also has known NAT-traversal issues due to its reliance on the GRE protocol.
- **L2TP/IPsec** — stronger security than PPTP, but depends on a single shared pre-shared key across all users, and can have its own NAT-traversal complications.
- **SSTP** — uses SSL/TLS over TCP 443, giving it strong firewall/NAT compatibility (same port as normal HTTPS traffic); Windows-only.
- **IKEv2** — generally considered the strongest current built-in option, especially for mobile/laptop users due to fast, seamless reconnection when network conditions change.

**Real-world context:** most modern enterprises (including my own workplace) have moved away from these traditional built-in Windows VPN protocols toward dedicated VPN/security platforms (e.g., Palo Alto GlobalProtect), or toward Zero Trust Network Access models that grant access to specific applications rather than full network-level access. This lab's RRAS setup was valuable for understanding the underlying tunneling/authentication concepts that these commercial platforms are built on top of.

## Key takeaway

Both RDP and VPN troubleshooting followed the same systematic pattern used throughout this lab: rule out basic connectivity first, then check firewall rules, then check the specific service's own configuration — rather than assuming the most complex possible cause first.
