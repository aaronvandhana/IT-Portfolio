# Project 2 – Active Directory Home Lab
### Lab Setup & Build Documentation

**Author:** Aaron  
**Domain:** aaronlab.local  
**Date Completed:** May 5, 2026  
**Platform:** Oracle VirtualBox  

---

## Overview

This project documents the build and configuration of a minimal Active Directory (AD) home lab using Oracle VirtualBox. The goal was to simulate a realistic help desk environment by setting up a Windows Server 2022 Domain Controller and a Windows 11 Pro client machine joined to the domain.

Active Directory is one of the most widely used identity and access management systems in enterprise IT environments. As a help desk technician, you will interact with AD on a daily basis — resetting passwords, unlocking accounts, creating users for new hires, disabling accounts during offboarding, and managing group memberships. This lab was built specifically to practice and document those tasks in a controlled, beginner-friendly environment.

**Lab Environment Summary:**

| Component | Details |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Domain Controller OS | Windows Server 2022 Standard Evaluation (Desktop Experience) |
| Client OS | Windows 11 Pro |
| Domain Name | aaronlab.local |
| DC IP Address | 192.168.100.10 |
| Client IP Address | 192.168.100.20 |
| Network Type | VirtualBox Internal Network (adlab) + NAT |

**Skills Demonstrated:**
- Deploying Windows Server 2022 in a virtualized environment
- Installing and configuring Active Directory Domain Services (AD DS)
- Promoting a server to a Domain Controller
- Configuring static IP addresses and DNS for domain communication
- Creating and managing domain user accounts
- Joining a Windows 11 client to an Active Directory domain
- Configuring Group Policy (Account Lockout Policy)
- Executing and documenting five realistic help desk ticket scenarios

---

## Phase 1 – Downloading Windows Server 2022

The first step was downloading the Windows Server 2022 evaluation ISO directly from Microsoft's Evaluation Center. The evaluation version is free for 180 days and includes all features needed for this lab.

> **IMG: 01_WinServer22_ISODownload**  
> *Microsoft Evaluation Center — Windows Server 2022 ISO download page. The English (United States) 64-bit ISO edition was selected.*

The ISO was saved locally and used as the installation media for the AD-Server virtual machine. No product key is required for the evaluation version — it activates automatically on first boot.

---

## Phase 2 – Creating the AD-Server Virtual Machine

With the ISO downloaded, the next step was creating a new virtual machine in VirtualBox configured to run Windows Server 2022.

> **IMG: 02_VM_ADServerVM_Setup**  
> *VirtualBox New Virtual Machine wizard — VM named "AD-Server", ISO attached, OS detected as Windows Server 2022 (64-bit). The unattended installation option was noted but bypassed to allow manual OS edition selection.*

> **IMG: 03_VM_ADServerVM_Setup02**  
> *Hardware configuration screen — Base Memory set to 4096 MB (4GB) and 2 CPUs allocated. Disk size set to 60GB. These are the minimum recommended specs for running Windows Server 2022 with Active Directory Domain Services.*

4GB of RAM is the recommended minimum for a Domain Controller. Running below this threshold can cause sluggish performance during AD operations and domain joins.

---

## Phase 3 – Installing Windows Server 2022

After the VM was created and the ISO attached, the installation began. During the OS selection screen, the correct edition was chosen carefully.

> **IMG: 04_VM-ADServerVM_OSinstallation**  
> *Windows Server 2022 edition selection screen — "Windows Server 2022 Standard Evaluation (Desktop Experience)" was selected. This edition installs the full GUI environment, which is required to use Server Manager, Active Directory Users and Computers, and Group Policy Management.*

The "Desktop Experience" option is critical — the non-GUI (Core) version would not provide the graphical tools needed for this lab.

> **IMG: 05_VM_ADServerVM_AdminPassword**  
> *Administrator account setup — a strong password was set for the built-in Administrator account. This password is used to log into the server and later to authenticate domain join requests from the client VM.*

After setup completed, the server rebooted and the Administrator account was ready for use.

---

## Phase 4 – Configuring Network Adapters

Before installing Active Directory, the server's network adapters needed to be configured. The lab uses two adapters per VM:

- **Adapter 1 (NAT):** Provides internet access for Windows Updates
- **Adapter 2 (Internal Network – "adlab"):** Allows the server and client VMs to communicate with each other in an isolated lab network

A static IP address was assigned to the Internal Network adapter on the server so that the client VM could reliably locate it for DNS and domain authentication.

> **IMG: 06_ADServerVM_Adapter2_IPSetup**  
> *IPv4 Properties for the Internal Network adapter — static IP configured as follows:*
> - IP Address: `192.168.100.10`
> - Subnet Mask: `255.255.255.0`
> - Default Gateway: *(blank)*
> - Preferred DNS Server: `127.0.0.1` *(server points to itself)*

Setting DNS to `127.0.0.1` tells the server to use its own DNS service — which is installed automatically when Active Directory is promoted. This is a required configuration step.

> **IMG: 07_ADServerVM_AdapterRenaming**  
> *Network Connections panel — adapters renamed to "Host Internet" (NAT) and "Lab" (Internal Network) for easy identification. The "Lab" adapter correctly shows "Unidentified network" — this is expected behavior since the Internal Network has no router or DHCP server.*

---

## Phase 5 – Windows Updates

Before promoting the server to a Domain Controller, Windows Updates were run to ensure the server was fully patched. Running updates before installing roles reduces the risk of installation issues caused by outdated system files.

> **IMG: 08_ADServerVM_WinUPdates**  
> *Windows Update screen on the AD-Server VM — several updates were downloading including cumulative updates for .NET Framework and the Windows Server operating system. Updates were allowed to complete before proceeding.*

---

## Phase 6 – Installing Active Directory Domain Services (AD DS)

With the server configured and updated, the next step was installing the Active Directory Domain Services role using Server Manager.

> **IMG: 09_ADServerVM_ServerManager**  
> *Server Manager Dashboard — the "Manage" menu is open with "Add Roles and Features" selected. This is the starting point for installing the AD DS role.*

> **IMG: 10_ADServerVM_ADInstallation**  
> *Add Roles and Features Wizard — "Role-based or feature-based installation" selected. This is the correct option for adding AD DS to a single server.*

> **IMG: 11_ADServerVM_ADInstallation02**  
> *AD DS dependency prompt — Server Manager detected that additional features are required for Active Directory Domain Services, including Group Policy Management, Remote Server Administration Tools, and AD DS/AD LDS Tools. "Add Features" was clicked to include all required dependencies.*

After clicking through the wizard and confirming the installation, the AD DS binaries were installed on the server. However, at this point the server is not yet a Domain Controller — that requires a separate promotion step.

---

## Phase 7 – Promoting the Server to a Domain Controller

After AD DS installed, a notification flag appeared in Server Manager prompting post-deployment configuration. This is where the server is promoted to an actual Domain Controller and the domain is created.

> **IMG: 12_ADServerVM_PromoteToDomainController**  
> *Server Manager post-deployment notification — the flag icon shows "Configuration required for Active Directory Domain Services." The "Promote this server to a domain controller" link was clicked to begin the promotion wizard.*

> **IMG: 13_ADServerVM_AddNewForest**  
> *Active Directory Domain Services Configuration Wizard — "Add a new forest" was selected since this is a brand new domain. The Root domain name was set to `aaronlab.local`.*

The `.local` suffix is a standard convention for internal lab and private domains — it is not resolvable on the public internet, which is intentional for a lab environment.

After completing the wizard, a DSRM (Directory Services Restore Mode) password was set and the server rebooted automatically to finalize the promotion.

> **IMG: 14_ADServerVM_NetBios_SignIn**  
> *Server login screen after reboot — the domain is now active. The login prompt shows `AARONLAB\Administrator`, confirming the server has successfully been promoted to a Domain Controller for the `aaronlab.local` domain.*

---

## Phase 8 – Creating Domain User Accounts

With the Domain Controller running, the next step was creating test user accounts in Active Directory Users and Computers (ADUC). These accounts are used to simulate real help desk ticket scenarios.

> **IMG: 15_ADServerVM_ActiveDirectorySearch**  
> *Start menu search for "Active Directory Users and Computers" — this is the primary tool used by help desk technicians to manage domain accounts, groups, and organizational units.*

> **IMG: 16_ADServerVM_AD_AddUsers**  
> *Active Directory Users and Computers — right-clicking the Users container to create a new User object. This is the standard method for provisioning new domain accounts.*

Three user accounts were created for the lab scenarios:

> **IMG: 17_ADServerVM_User_jsmith**  
> *New User wizard — account created for John Smith with logon name `jsmith@aaronlab.local`. This account will be used in Ticket #001 (account lockout) and Ticket #004 (account disable).*

> **IMG: 18_ADServerVM_User_mlopez**  
> *New User wizard — account created for Marla Lopez with logon name `mlopez@aaronlab.local`. This account will be used in Ticket #002 (password reset) and Ticket #005 (group membership update).*

> **IMG: 19_ADServerVM_User_helpdesk**  
> *New User wizard — account created for Help Desk Tech with logon name `help.desk@aaronlab.local`. This account represents the help desk technician role used for administrative tasks.*

All accounts were created with temporary passwords and the "User must change password at next logon" option, following standard security best practices for new account provisioning.

---

## Phase 9 – Creating the Windows 11 Client VM

With the Domain Controller and user accounts ready, a second virtual machine was created to serve as the domain-joined workstation. This simulates the end-user machine that a help desk technician would support.

> **IMG: 20_ADClientVM_Setup**  
> *VirtualBox New Virtual Machine wizard for the client — VM named "AD-Client", Windows 11 Pro ISO attached. Both network adapters are visible in the summary: Adapter 1 (NAT) and Adapter 2 (Internal Network, "adlab"). Windows 11 Pro was selected specifically because the Home edition does not support domain join.*

---

## Phase 10 – Configuring the Client's Network and Verifying Connectivity

The client VM was configured with a static IP on its Internal Network adapter, with DNS pointing to the Domain Controller.

> **IMG: 21_ADClientVM_IPSettup**  
> *Client network settings and ping test — static IP set to `192.168.100.20`, DNS set to `192.168.100.10`. The PowerShell ping test shows 4 packets sent, 4 received, 0% packet loss — confirming the two VMs can communicate over the Internal Network adapter.*

This connectivity test is a critical prerequisite before attempting to join the domain. A failed ping at this stage indicates a networking misconfiguration that must be resolved before proceeding.

---

## Phase 11 – Joining the Client to the Domain

With connectivity confirmed, the Windows 11 client was joined to the `aaronlab.local` domain.

> **IMG: 22_ADClientVM_DomainChange**  
> *System Properties — Computer Name/Domain Changes dialog showing `aaronlab.local` entered as the domain. Administrator credentials (`AARONLAB\Administrator`) were entered in the Windows Security prompt to authorize the join.*

> **IMG: 23_ADClientVM_DomainChangeSuccess**  
> *Domain join success message — "Welcome to the aaronlab.local domain." The machine was restarted to complete the domain join process.*

After rebooting, the client is now a member of the `aaronlab.local` domain. Domain users can log in at the "Other user" prompt using their `AARONLAB\username` credentials.

---

## Phase 12 – Configuring Account Lockout Policy

Before running the ticket scenarios, a Group Policy configuration issue was discovered and resolved. By default, Active Directory does not enforce an account lockout policy — failed login attempts continue indefinitely without locking the account.

To simulate realistic help desk conditions, the Account Lockout Policy was configured via Group Policy Management.

> **IMG: 26_ADServerVM_GroupPolicyManagementUpdate**  
> *Group Policy Management Editor — Default Domain Policy, Account Lockout Policy settings configured:*
> - Account lockout duration: **30 minutes**
> - Account lockout threshold: **5 invalid logon attempts**
> - Allow Administrator account lockout: **Enabled**
> - Reset account lockout counter after: **30 minutes**

After saving the policy, `gpupdate /force` was run on the client VM to apply the new settings immediately rather than waiting for the standard policy refresh interval.

> **Note for documentation:** This configuration step was not part of the original plan but was discovered as a necessary troubleshooting step. Identifying and resolving this gap demonstrates real-world diagnostic thinking. See `troubleshooting-log.md` for the full entry.

---

## Lab Build Complete

At this point the lab environment was fully operational:

- ✅ Windows Server 2022 Domain Controller running `aaronlab.local`
- ✅ Three domain user accounts created (jsmith, mlopez, help.desk)
- ✅ Windows 11 Pro client joined to the domain
- ✅ Both VMs communicating over the Internal Network adapter
- ✅ Account Lockout Policy configured via Group Policy
- ✅ Domain logins verified from the client workstation

The following section documents all five help desk ticket scenarios performed in this environment. For full ticket documentation including resolutions and verification steps, see `ticket-scenarios.md`.

---

## Ticket Scenario Screenshots

### Pre-Ticket: Initial Domain Login Test

> **IMG: 24_ADClientVM_LoginUser_jsmith**  
> *Client VM login screen — "Other user" selected and `aaronlab\jsmith` entered to test domain authentication for the first time. This confirmed that domain accounts created on the server were accessible from the client workstation.*

---

### Ticket #001 – Account Lockout (John Smith)

To simulate a real lockout scenario, the wrong password was intentionally entered five or more times for jsmith on the client VM.

> **IMG: 25_ADClientVM_User_jsmith_loginfail**  
> *Client VM — "Invalid credentials, delaying next attempt..." message appearing during repeated failed login attempts. At this point the lockout policy had not yet been applied.*

After configuring and applying the Account Lockout Policy via Group Policy:

> **IMG: 27_ADClientVM_User_jsmith_lockout**  
> *Client VM — "The referenced account is currently locked out and may not be logged on to." This confirms the lockout policy is working correctly. This screenshot serves as the "before" state for Ticket #001.*

> **IMG: 28_ADServerVM_unlockuser_jsmith**  
> *Server VM — Active Directory Users and Computers, jsmith Properties, Account tab. The "Unlock account: This account is currently locked out on this Active Directory Domain Controller" checkbox is checked. The checkbox was unchecked and Apply was clicked to unlock the account.*

> **IMG: 29_ADClientVM_user_jsmith_loginsuccess**  
> *Client VM — "Welcome" screen for John Smith following a successful login after the account was unlocked. This confirms the ticket was resolved successfully.*

---

### Ticket #002 – Password Reset (Marla Lopez)

> **IMG: 30_ADServerVM_user_mlopez_passreset**  
> *Server VM — Reset Password dialog for mlopez. A temporary password was set with "User must change password at next logon" checked. Account Lockout Status shows "Unlocked."*

> **IMG: 31_ADClientVM_user_mlopez_newpassword**  
> *Client VM — "The user's password must be changed before signing in." This confirms the forced password change policy is working as expected.*

> **IMG: 32_ADClientVM_user_mlopez_passwordchanged**  
> *Client VM — "Your password has been changed." Marla Lopez successfully set a new password and the login completed. Ticket resolved.*

---

### Ticket #003 – New Employee Onboarding (Alex Johnson)

> **IMG: 33_ADServerVM_create_user_ajohnson_01**  
> *Server VM — New Object - User wizard for Alex Johnson. First name: Alex, Last name: Johnson, logon name: `ajohnson@aaronlab.local`.*

> **IMG: 34_ADServerVM_create_user_ajohnson_02**  
> *Password configuration for ajohnson — temporary password set with "User must change password at next logon" checked.*

> **IMG: 35_ADServerVM_create_user_ajohnson03**  
> *Alex Johnson Properties — Select Groups dialog with IT-Staff entered. The account was added to the IT-Staff security group as requested by HR.*

---

### Ticket #004 – Employee Offboarding / Account Disable (John Smith)

> **IMG: 36_ADServerVM_disable_user_jsmith**  
> *Server VM — Right-click context menu on jsmith with "Disable Account" highlighted. The status bar at the bottom confirms "Disables the account for the current selection."*

> **IMG: 37_ADServerVM_disabled_user_jsmith**  
> *Server VM — Disabled-Users OU selected in the left panel, with John Smith's account now residing inside it. The account icon shows the disabled state indicator. The account was moved to this OU to keep it separate from active users while retaining it per the 30-day retention policy.*

> **IMG: 38_ADClientVM_user_jsmith_disabled_account**  
> *Client VM — "Your account has been disabled. Please see your system administrator." Attempted login from the client confirmed the account is inaccessible. Ticket resolved and verified.*

---

### Ticket #005 – Group Membership Update (Marla Lopez)

> **IMG: 39_ADClientVM_user_mlopez_manager**  
> *Server VM — Maria Lopez Properties, Member Of tab. Both "Domain Users" and "Managers" are listed, confirming mlopez was successfully added to the Managers security group. Ticket resolved.*

---

## Summary

This lab successfully demonstrated a complete Active Directory environment from initial build through real-world help desk task execution. All five ticket scenarios were completed, documented, and verified with before-and-after screenshots.

For full ticket documentation, see [`ticket-scenarios.md`](ticket-scenarios.md).  
For issues encountered during the build, see [`troubleshooting-log.md`](troubleshooting-log.md).
