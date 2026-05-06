# Troubleshooting Log – Active Directory Home Lab

**Project:** Project 2 – Active Directory Home Lab  
**Domain:** aaronlab.local  
**Technician:** Aaron Vandhana  
**Date:** May 5, 2026  

---

## Overview

This log documents every real issue encountered during the build and configuration of the Active Directory home lab. Each entry includes the problem observed, the diagnostic steps taken, the root cause identified, and the resolution applied. This log demonstrates real troubleshooting methodology — not just a list of steps that worked, but an honest record of what went wrong and how it was fixed.

---

## Issue Index

| # | Issue | Component | Status |
|---|---|---|---|
| 001 | ISO failed to load — "Cannot find Microsoft Software License Terms" | AD-Server VM | ✅ Resolved |
| 002 | VM failed to boot — "No bootable option or device was found" | AD-Server VM | ✅ Resolved |
| 003 | Client VM could not reach server — network not configured | VirtualBox Networking | ✅ Resolved |
| 004 | Account lockout not triggering after failed logins | Group Policy | ✅ Resolved |
| 005 | Disabled account (jsmith) still allowed login on first attempt | Windows Cached Credentials | ✅ Resolved |

---

## ISSUE #001 – ISO Failed to Load During Server Installation

**Date:** 05/05/2026  
**Affected Component:** AD-Server Virtual Machine  
**Severity:** Blocker  
**Status:** Resolved  

---

**Problem Observed:**

Immediately after starting the AD-Server VM for the first time, a dialog appeared before the Windows setup screen could load:

> *"Windows cannot find the Microsoft Software License Terms. Make sure the installation sources are valid and restart the installation."*

Clicking OK dismissed the error but the VM could not proceed with installation.

---

**Diagnostic Steps:**

1. Dismissed the error and powered off the VM
2. Opened VM Settings → Storage to inspect how the ISO was attached
3. Checked that the ISO file existed on the host machine and was not corrupted
4. Verified the file size of the downloaded ISO — a valid Windows Server 2022 ISO should be approximately 4.7–5.2 GB

---

**Root Cause:**

The ISO was not properly attached to the VM's optical drive in VirtualBox. The VM was created with the ISO path referenced, but VirtualBox was not reading it correctly at boot time.

---

**Resolution:**

1. Powered off the AD-Server VM completely
2. Opened Settings → Storage
3. Clicked the disc icon under the IDE Controller
4. Selected **"Choose a disk file"** and reselected the Windows Server 2022 ISO
5. Clicked OK to save the settings
6. Restarted the VM

On restart, the Windows Server setup loaded correctly and proceeded to the OS edition selection screen.

---

**Lesson Learned:**

Always verify ISO attachment in Storage settings before starting a new VM. VirtualBox sometimes references the ISO path without correctly mounting it as bootable media, particularly if the ISO was attached during the VM creation wizard rather than through Storage settings afterward.

---

## ISSUE #002 – VM Failed to Boot — "No Bootable Option or Device Was Found"

**Date:** 05/05/2026  
**Affected Component:** AD-Server Virtual Machine  
**Severity:** Blocker  
**Status:** Resolved  

---

**Problem Observed:**

After attempting to fix Issue #001, the VM booted and displayed:

> *"Press any key to boot from CD or DVD......"*
> *"BdsDxe: No bootable option or device was found."*
> *"BdsDxe: Press any key to enter the Boot Manager Menu."*

A VirtualBox dialog also appeared stating the VM failed to boot, with a DVD dropdown showing `<not selected>`.

---

**Diagnostic Steps:**

1. Recognized the "Press any key to boot from CD or DVD" prompt — this message appears for only about 3 seconds before the system skips the ISO
2. Identified that the ISO was still not being recognized as the boot device
3. Used the VirtualBox dialog's DVD dropdown to reattach the ISO while the VM was running

---

**Root Cause:**

Two separate causes contributed to this issue:

1. The ISO was not being saved as a persistent attachment in the VM's storage settings — it was dropping on each reboot
2. The "Press any key to boot from CD or DVD" prompt was being missed because it disappears very quickly — if no key is pressed in time, the VM skips the ISO entirely and attempts to boot from the (empty) hard disk

---

**Resolution:**

1. While the VirtualBox boot failure dialog was still open, clicked the **DVD dropdown → Choose a disk file** and selected the ISO
2. Clicked **"Mount and Retry Boot"**
3. Immediately clicked inside the VM window when "Press any key to boot from CD or DVD......" appeared and pressed the Space bar
4. Windows Server setup loaded successfully

To prevent recurrence, also confirmed the ISO was permanently saved in Settings → Storage → IDE Controller so it would be available on every boot.

---

**Lesson Learned:**

The "Press any key" boot prompt has a very short window — approximately 2–3 seconds. Clicking inside the VM window first to capture mouse/keyboard input, then immediately pressing a key, is essential. Missing this prompt causes the VM to attempt booting from the hard disk, which fails because no OS is installed yet.

---

## ISSUE #003 – Client VM Could Not Communicate With the Server

**Date:** 05/05/2026  
**Affected Component:** VirtualBox Network Configuration  
**Severity:** Blocker  
**Status:** Resolved  

---

**Problem Observed:**

After the AD-Server and AD-Client VMs were both running, the client could not reach the server. The Lab (Internal Network) adapter on both VMs showed "Unidentified network" in Network Connections, and initial ping tests were failing.

---

**Diagnostic Steps:**

1. Opened Network Connections on both VMs to identify which adapter was which
2. Checked VirtualBox settings for both VMs to confirm Adapter 2 was set to Internal Network
3. Verified the Internal Network name matched exactly on both VMs
4. Checked whether static IPs had been assigned to the correct adapters
5. Ran `ping 192.168.100.10` from the client to test connectivity

---

**Root Cause:**

The static IP addresses had not yet been configured on the Internal Network (Lab) adapters. Both VMs had the correct VirtualBox network type set, but without static IPs assigned, the adapters had no addresses and could not communicate.

Additionally, the "Unidentified network" status on the Lab adapter caused initial concern — this was later confirmed to be expected and normal behavior. The Internal Network adapter has no router or gateway, so Windows cannot identify it as a known network type. This does not affect functionality.

---

**Resolution:**

Configured static IPv4 settings on the Internal Network adapter of each VM:

**AD-Server (Lab adapter):**
- IP Address: `192.168.100.10`
- Subnet Mask: `255.255.255.0`
- Default Gateway: *(blank)*
- Preferred DNS: `127.0.0.1`

**AD-Client (Lab adapter):**
- IP Address: `192.168.100.20`
- Subnet Mask: `255.255.255.0`
- Default Gateway: *(blank)*
- Preferred DNS: `192.168.100.10`

After applying these settings, ran `ping 192.168.100.10` from the client:

```
Pinging 192.168.100.10 with 32 bytes of data:
Reply from 192.168.100.10: bytes=32 time=1ms TTL=128
Reply from 192.168.100.10: bytes=32 time<1ms TTL=128
Reply from 192.168.100.10: bytes=32 time<1ms TTL=128
Reply from 192.168.100.10: bytes=32 time=4ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

Connectivity confirmed. Domain join proceeded successfully after this point.

---

**Lesson Learned:**

"Unidentified network" on an Internal Network adapter is normal and not an error. The real connectivity test is always `ping` — not the Windows network status indicator. DNS configuration on the client is equally critical: the client must point its DNS to the Domain Controller's IP (`192.168.100.10`), not to a public DNS server, otherwise domain join will fail with a "domain not found" error even if ping succeeds.

---

## ISSUE #004 – Account Lockout Policy Not Enforced by Default

**Date:** 05/05/2026  
**Affected Component:** Active Directory Group Policy  
**Severity:** Medium  
**Status:** Resolved  

---

**Problem Observed:**

While setting up Ticket #001 (account lockout scenario for jsmith), repeated failed login attempts on the client VM produced the message:

> *"Invalid credentials, delaying next attempt..."*

The account was never locked. After five, six, seven, and more failed attempts, the account remained accessible with the correct password. The lockout behavior described in Ticket #001 could not be demonstrated.

---

**Diagnostic Steps:**

1. Confirmed the failed login attempts were being entered correctly — the error message was genuine, not a typo issue
2. Researched default Active Directory behavior — discovered that AD does not enforce an Account Lockout Policy by default
3. Located the Group Policy Management console on the Domain Controller
4. Navigated to: Default Domain Policy → Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy
5. Confirmed all lockout settings were showing "Not Defined"

---

**Root Cause:**

Active Directory Domain Services installs with no account lockout policy configured. This is a known default — Microsoft leaves lockout policy unconfigured so administrators can define their own thresholds appropriate to their environment. In a production environment, setting this policy would be one of the first security configurations applied after domain creation.

---

**Resolution:**

Configured the Account Lockout Policy in Group Policy Management on the Domain Controller:

- **Account lockout threshold:** 5 invalid logon attempts
- **Account lockout duration:** 30 minutes
- **Reset account lockout counter after:** 30 minutes
- **Allow Administrator account lockout:** Enabled

Ran `gpupdate /force` on the client VM to apply the new policy immediately.

Tested by entering an incorrect password for jsmith five times. After the fifth attempt, the client displayed:

> *"The referenced account is currently locked out and may not be logged on to."*

Policy confirmed working. Ticket #001 proceeded successfully.

---

**Lesson Learned:**

Default Active Directory configurations are not the same as secure configurations. Several important security policies — including account lockout, password complexity, and audit logging — require manual configuration by an administrator. Discovering and resolving this gap during the lab demonstrates the kind of diagnostic thinking that is valuable in a real help desk or sysadmin role.

This finding was documented as an additional note in the Ticket #001 entry in `ticket-scenarios.md`.

---

## ISSUE #005 – Disabled Account Still Allowed Login (Cached Credentials)

**Date:** 05/05/2026  
**Affected Component:** Windows 11 Client — Credential Caching  
**Severity:** Low  
**Status:** Resolved  

---

**Problem Observed:**

After completing Ticket #004 (disabling jsmith's account on the Domain Controller), jsmith was still able to log into the client VM successfully. The account appeared to be disabled in Active Directory, but the client workstation allowed the login anyway.

---

**Diagnostic Steps:**

1. Confirmed jsmith's account was disabled in Active Directory Users and Computers — the account showed the disabled icon and was located in the Disabled-Users OU
2. Attempted login on the client VM as jsmith — login succeeded unexpectedly
3. Researched Windows credential caching behavior — identified that Windows caches the last N successful domain logins locally by default
4. Recognized that jsmith had previously logged in successfully on this machine, so cached credentials existed

---

**Root Cause:**

Windows caches domain credentials locally to allow login when the domain controller is unreachable (e.g., no network connection). When a user logs in, Windows first checks the local cache. If cached credentials match, login succeeds regardless of the account's current status on the Domain Controller.

This is intentional Windows behavior — it is not a bug. However, it means that disabling an account on the DC does not immediately prevent access on a machine where the user has previously authenticated if the machine checks the cache first.

---

**Resolution:**

Signed out of jsmith's session completely (not just locked the screen). On the next login attempt, the client had no active session to resume and was forced to authenticate directly against the Domain Controller, which returned the disabled account status.

The client then displayed:

> *"Your account has been disabled. Please see your system administrator."*

Ticket #004 verification confirmed successful.

---

**Lesson Learned:**

In a real help desk environment, when disabling an account for a departing employee, you should also:

1. Sign out any active sessions on their workstation
2. If possible, lock or shut down their workstation to clear the session entirely
3. In high-security situations, consider revoking Kerberos tickets (using `klist purge` or restarting the Netlogon service) to force immediate re-authentication

Cached credentials are a useful feature for remote workers but can give a false impression that account changes took effect when they have not yet been enforced on the local machine.

---

## Summary

| Issue | Root Cause | Resolution |
|---|---|---|
| ISO load failure | ISO not properly mounted in VirtualBox storage settings | Reattached ISO via Settings → Storage |
| VM boot failure | Missed the "Press any key" prompt / ISO not persistent | Mounted ISO via VirtualBox dialog, pressed key immediately |
| No network connectivity | Static IPs not configured on Internal Network adapters | Assigned static IPs to Lab adapters on both VMs |
| Lockout policy not working | AD DS does not configure lockout policy by default | Configured Account Lockout Policy via Group Policy Management |
| Disabled account still accessible | Windows cached credentials allowed login without DC check | Fully signed out user before testing — forced live DC authentication |

---

*This troubleshooting log was written as part of Project 2 – Active Directory Home Lab. All issues documented here were encountered and resolved during the actual build process.*
