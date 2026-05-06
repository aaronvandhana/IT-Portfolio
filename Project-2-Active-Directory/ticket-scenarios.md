# Ticket Scenarios – Active Directory Help Desk Lab

**Project:** Project 2 – Active Directory Home Lab  
**Domain:** aaronlab.local  
**Technician:** Aaron Vandhana  
**Date:** May 5, 2026  

---

## Overview

This document contains five help desk ticket scenarios performed in the Active Directory lab environment. Each ticket simulates a real-world request that a help desk or desktop support technician would handle on a daily basis. All tasks were performed on a Windows Server 2022 Domain Controller managing the `aaronlab.local` domain, with a Windows 11 Pro client machine used to verify results.

---

## Ticket Index

| # | Issue | User | Priority | Status |
|---|---|---|---|---|
| 001 | Account Lockout | John Smith (jsmith) | High | ✅ Resolved |
| 002 | Password Reset | Marla Lopez (mlopez) | Medium | ✅ Resolved |
| 003 | New Employee Onboarding | Alex Johnson (ajohnson) | Medium | ✅ Resolved |
| 004 | Employee Offboarding – Account Disable | John Smith (jsmith) | High | ✅ Resolved |
| 005 | Security Group Membership Update | Marla Lopez (mlopez) | Low | ✅ Resolved |

---

## TICKET #001 – Account Lockout

**Date:** 05/05/2026  
**Submitted By:** John Smith  
**Technician:** Aaron Vandhana  
**Priority:** High  
**Status:** Resolved  

---

**Ticket Description:**

> "I tried logging into my computer this morning and got a message saying my account is locked out. I have an important meeting in 10 minutes and need access immediately."

---

**Background – Pre-Ticket Configuration:**

Before this ticket could be simulated, it was discovered that the default Active Directory configuration does not enforce an account lockout policy. Failed login attempts would generate "Invalid credentials" errors indefinitely without locking the account.

To resolve this, the Account Lockout Policy was configured via Group Policy Management on the Domain Controller:

- Navigate to: Group Policy Management → Forest: aaronlab.local → Domains → aaronlab.local → Default Domain Policy → Edit
- Path: Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy
- Settings applied:
  - Account lockout threshold: **5 invalid logon attempts**
  - Account lockout duration: **30 minutes**
  - Reset account lockout counter after: **30 minutes**
  - Allow Administrator account lockout: **Enabled**
- Ran `gpupdate /force` on the client VM to apply the policy immediately

This configuration step is documented in detail in `troubleshooting-log.md`.

---

**Steps to Reproduce Issue:**

1. On the client VM, clicked "Other user" at the login screen
2. Entered `aaronlab\jsmith` with an incorrect password
3. Repeated incorrect login attempts 5 times
4. Account locked — client displayed: *"The referenced account is currently locked out and may not be logged on to."*

**Screenshot:** `27_ADClientVM_User_jsmith_lockout.png`

---

**Troubleshooting Steps:**

1. Verified the user's identity and confirmed the reported issue
2. Opened **Active Directory Users and Computers** on the Domain Controller
3. Navigated to `aaronlab.local → Users`
4. Located **John Smith (jsmith)**
5. Right-clicked → **Properties → Account tab**
6. Confirmed the **"Unlock account: This account is currently locked out on this Active Directory Domain Controller"** checkbox was checked

**Screenshot:** `28_ADServerVM_unlockuser_jsmith.png`

---

**Resolution:**

Unchecked the "Unlock account" checkbox and clicked **Apply → OK**. Informed the user that their account had been unlocked and advised them to use the correct credentials going forward.

---

**Verification:**

Returned to the client VM and logged in as `aaronlab\jsmith` using the correct password. The "Welcome" screen appeared, confirming successful domain authentication and full resolution of the issue.

**Screenshot:** `29_ADClientVM_user_jsmith_loginsuccess.png`

**Time to Resolve:** ~8 minutes (including Group Policy configuration)

---

## TICKET #002 – Password Reset

**Date:** 05/05/2026  
**Submitted By:** Marla Lopez  
**Technician:** Aaron Vandhana  
**Priority:** Medium  
**Status:** Resolved  

---

**Ticket Description:**

> "I forgot my password over the weekend and can't log in. I've tried everything I can remember. Can you please reset it for me?"

---

**Troubleshooting Steps:**

1. Verified the user's identity before proceeding with any account changes
2. Opened **Active Directory Users and Computers** on the Domain Controller
3. Navigated to `aaronlab.local → Users`
4. Located **Marla Lopez (mlopez)**
5. Right-clicked → **Reset Password**
6. Set a temporary password
7. Checked **"User must change password at next logon"** to enforce a password change on first login
8. Confirmed Account Lockout Status showed **"Unlocked"** — no lockout issue was present

**Screenshot:** `30_ADServerVM_user_mlopez_passreset.png`

---

**Resolution:**

Temporary password was set and communicated to the user through a secure channel. The "User must change password at next logon" flag ensures the user sets their own private password immediately, maintaining account security.

---

**Verification:**

Logged into the client VM as `aaronlab\mlopez` using the temporary password. The system displayed: *"The user's password must be changed before signing in."*

**Screenshot:** `31_ADClientVM_user_mlopez_newpassword.png`

After setting a new password, the system displayed: *"Your password has been changed."* — confirming the reset was successful and the account is fully accessible.

**Screenshot:** `32_ADClientVM_user_mlopez_passwordchanged.png`

**Time to Resolve:** ~5 minutes

---

## TICKET #003 – New Employee Onboarding

**Date:** 05/05/2026  
**Submitted By:** HR Department  
**Technician:** Aaron Vandhana  
**Priority:** Medium  
**Status:** Resolved  

---

**Ticket Description:**

> "We have a new hire starting Monday — Alex Johnson, joining the IT Department. Please create their domain account and ensure they have access to IT resources. They will need to set their own password on first login."

---

**Troubleshooting Steps:**

1. Opened **Active Directory Users and Computers** on the Domain Controller
2. Navigated to `aaronlab.local → Users`
3. Right-clicked the Users container → **New → User**
4. Filled in the following details:
   - First name: **Alex**
   - Last name: **Johnson**
   - Full name: **Alex Johnson**
   - User logon name: **ajohnson**
   - Domain: **@aaronlab.local**

**Screenshot:** `33_ADServerVM_create_user_ajohnson_01.png`

5. On the password screen:
   - Set a temporary password
   - Checked **"User must change password at next logon"**
   - Left "Account is disabled" unchecked so the account is active immediately

**Screenshot:** `34_ADServerVM_create_user_ajohnson_02.png`

6. After account creation, opened **ajohnson Properties → Member Of tab → Add**
7. Typed **IT-Staff** in the object name field → clicked **Check Names** → clicked **OK**

**Screenshot:** `35_ADServerVM_create_user_ajohnson03.png`

---

**Resolution:**

Domain account created for Alex Johnson (`ajohnson@aaronlab.local`) with a temporary password and forced password change on first login. Account was added to the IT-Staff security group as requested by HR. Temporary credentials were communicated to HR for new hire orientation.

---

**Verification:**

Opened ajohnson's Properties → Member Of tab and confirmed **IT-Staff** was listed as a group membership alongside Domain Users. Account status confirmed as active and ready for first login.

**Time to Resolve:** ~7 minutes

---

## TICKET #004 – Employee Offboarding / Account Disable

**Date:** 05/05/2026  
**Submitted By:** HR Department  
**Technician:** Aaron Vandhana  
**Priority:** High  
**Status:** Resolved  

---

**Ticket Description:**

> "John Smith has resigned effective today. Please disable his domain account immediately per company security policy. Do NOT delete the account — we need to retain it for 30 days for compliance and access review purposes."

---

**Troubleshooting Steps:**

1. Opened **Active Directory Users and Computers** on the Domain Controller
2. Navigated to `aaronlab.local → Users`
3. Located **John Smith (jsmith)**
4. Right-clicked → **Disable Account**
   - Status bar confirmed: *"Disables the account for the current selection."*

**Screenshot:** `36_ADServerVM_disable_user_jsmith.png`

5. To keep the Users container organized and clearly flag the account as inactive:
   - Created a new Organizational Unit: right-clicked `aaronlab.local` → **New → Organizational Unit** → named it **Disabled-Users**
   - Moved jsmith into the Disabled-Users OU by right-clicking → **Move**

**Screenshot:** `37_ADServerVM_disabled_user_jsmith.png`

---

**Resolution:**

Account disabled and moved to the Disabled-Users OU for tracking. The account remains in Active Directory for the 30-day retention period per company policy, but cannot be used to authenticate to any domain resources.

---

**Verification:**

Signed out of the client VM and attempted to log in as `aaronlab\jsmith`. The system displayed: *"Your account has been disabled. Please see your system administrator."* — confirming the account is fully inaccessible.

**Screenshot:** `38_ADClientVM_user_jsmith_disabled_account.png`

**Note:** On the first attempt, jsmith was still able to log in due to Windows cached credentials. Signing out fully and attempting a fresh login — which forces the client to check with the Domain Controller — produced the correct disabled account message.

**Time to Resolve:** ~5 minutes

---

## TICKET #005 – Security Group Membership Update

**Date:** 05/05/2026  
**Submitted By:** Department Manager  
**Technician:** Aaron Vandhana  
**Priority:** Low  
**Status:** Resolved  

---

**Ticket Description:**

> "Marla Lopez has been promoted to team lead and will now be managing the department. She needs to be added to the Managers security group so she has the appropriate level of access going forward."

---

**Troubleshooting Steps:**

1. Opened **Active Directory Users and Computers** on the Domain Controller
2. Navigated to `aaronlab.local → Users`
3. Confirmed the **Managers** security group existed — created it if not present:
   - Right-click Users container → **New → Group**
   - Group name: **Managers**
   - Group scope: **Global**
   - Group type: **Security**
4. Located **Marla Lopez (mlopez)**
5. Right-clicked → **Properties → Member Of tab → Add**
6. Typed **Managers** in the object name field → clicked **OK**
7. Clicked **Apply** to save the group membership change

---

**Resolution:**

Marla Lopez (mlopez) was successfully added to the Managers security group. In a production environment, this group membership would grant her access to shared resources, folders, or applications restricted to the Managers group.

---

**Verification:**

Opened mlopez Properties → Member Of tab and confirmed both **Domain Users** and **Managers** were listed as active group memberships.

**Screenshot:** `39_ADClientVM_user_mlopez_manager.png`

**Time to Resolve:** ~3 minutes

---

## Lessons Learned

Working through these five ticket scenarios highlighted several important concepts that apply directly to real help desk work:

**Group Policy is not configured by default.** Active Directory installs with no account lockout policy enabled. In a production environment this would be one of the first things configured by a sysadmin, but discovering and resolving this gap reinforced how Group Policy works and why it matters for security.

**Cached credentials can mask account changes.** When jsmith's account was disabled, the client initially allowed login because Windows had cached the credentials locally. A fresh sign-out and sign-in forced a live check against the Domain Controller. This is important to understand when verifying account changes in a real environment.

**DNS is the foundation of Active Directory.** Every domain join failure, every "domain not found" error, and every authentication issue traces back to DNS. The client VM must point its DNS to the Domain Controller's IP address — without this, nothing works.

**Disabling vs. deleting accounts.** Company offboarding policy typically requires accounts to be disabled and retained for a period before deletion. Moving disabled accounts to a dedicated OU (Disabled-Users) keeps the directory organized and makes audits easier.

**Verification is part of every ticket.** Every resolution in this lab was followed by a live test — logging in from the client, checking group membership tabs, confirming error messages. Closing a ticket without verification is incomplete work.
