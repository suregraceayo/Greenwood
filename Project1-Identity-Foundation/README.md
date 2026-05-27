# 🏛 Project 1 — Identity Foundation: Microsoft Entra ID
**Greenwood Accountants | Microsoft 365 Identity & Security Implementation**


## 📋 Project Overview

| Field | Details |
| **Client** | Greenwood Accountants |
| **Difficulty** | Beginner |
| **Estimated Time** | 20–25 hours |
| **Target Roles** | M365 Administrator · IT Administrator · Identity Engineer · IAM Engineer |
| **Deadline Driver** | 60-day cyber insurance remediation window |


## 🏢 Business Scenario

Greenwood Accountants is migrating from on-premises Active Directory to Microsoft 365. The firm has 80 staff across two offices. Prior to this project, the IT manager had been creating accounts manually with no MFA in place. Following a failed cyber insurance assessment, the insurer issued a 60-day ultimatum to implement a secure identity foundation.


## 👥 1. User Accounts

### Naming Convention
Format: `firstname.lastname@greenwoodaccountants.co.uk`
Tie-breaker rule (for duplicate names): append middle initial or department suffix
e.g. `john.a.smith@greenwoodaccountants.co.uk` or `john.smith.finance@greenwoodaccountants.co.uk`

### Test Users Created (10 Users)

| # | Display Name | UPN | Department | Job Title | Manager |
|---|---|---|---|---|---|
| 1 | Emma greenwood | emma.greenwood@christtech.co.uk | Finance | Finance Manager | — |
| 2 | Daniel greenwood | daniel.greenwood@christtech.co.uk | Finance | Accounts Assistant | Emma greenwood |
| 3 | Fatima greenwood | fatima.greenwood@christtech.co.uk | HR | HR Manager | — |
| 4 | Michael greenwood | michael.greenwood@christtech.co.uk | HR | HR Coordinator | Fatima greenwood |
| 5 | Samuel greenwood | samuel.greenwood@christtech.co.uk | IT | IT Administrator | — |
| 6 | Ethan greenwood | ethan.greenwood@christtech.co.uk | IT | IT Support Analyst | Samuel greenwood |
| 7 | Grace greenwood | grace.greenwood@christtech.co.uk | Sales | Sales Manager | — |
| 8 | Joy greenwood | joy.greenwood@christtech.co.uk | Sales | Sales Executive | Grace greenwood |
| 9 | Henry greenwood | henry.greenwood@christtech.co.uk | Management | Operations Director | — |
| 10 | Kemi greenwood | kemi.greenwood@christtech.co.uk | Management | Executive Assistant | Henry greenwood |

### PowerShell — Create Users
```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"

$passwordProfile = @{
    Password = "TempPass2026!"
    ForceChangePasswordNextSignIn = $true
}

New-MgUser -DisplayName "Emma greenwood" `
  -UserPrincipalName "emma.greenwood@christtech.co.uk" `
  -MailNickname "emma.greenwood" `
  -Department "Finance" `
  -JobTitle "Finance Manager" `
  -AccountEnabled $true `
  -PasswordProfile $passwordProfile

# Repeat for all 10 users with relevant details
```


## 👥 2. Security Groups

### Group Structure

| Group Name | Type | Membership | Purpose |
|---|---|---|---|
| SG-Finance-Users | Security | Assigned | Finance department access |
| SG-HR-Users | Security | Assigned | HR department access |
| SG-IT-Admins | Security | Assigned | IT admin privileges |
| SG-Management | Security | Assigned | Management-level access |
| SG-All-Staff | Security | Dynamic | All enabled staff accounts |

### Dynamic Rule — SG-All-Staff
```powershell
(user.accountEnabled -eq true) and (user.userType -eq "Member")
```

### PowerShell — Create Security Groups
```powershell
# Assigned group example
New-MgGroup -DisplayName "SG-Finance-Users" `
  -MailNickname "SG-Finance-Users" `
  -SecurityEnabled $true `
  -MailEnabled $false `
  -GroupTypes @()

# Dynamic group
New-MgGroup -DisplayName "SG-All-Staff" `
  -MailNickname "SG-All-Staff" `
  -SecurityEnabled $true `
  -MailEnabled $false `
  -GroupTypes @("DynamicMembership") `
  -MembershipRule '(user.accountEnabled -eq true) and (user.userType -eq "Member")' `
  -MembershipRuleProcessingState "On"
```

### Add Members to Assigned Groups
```powershell
New-MgGroupMember -GroupId "<SG-Finance-Users-ID>" `
  -DirectoryObjectId "<user-id>"
```


## 📦 3. Microsoft 365 Groups

| Group Name | Purpose | Auto-Creates |
|---|---|---|
| M365-Finance | Finance collaboration | Teams · SharePoint · Mailbox |
| M365-HR | HR collaboration | Teams · SharePoint · Mailbox |
| M365-AllCompany | Company-wide comms | Teams · SharePoint · Mailbox |

### PowerShell — Create M365 Groups
```powershell
New-MgGroup -DisplayName "M365-Finance" `
  -MailNickname "M365-Finance" `
  -GroupTypes @("Unified") `
  -MailEnabled $true `
  -SecurityEnabled $false `
  -Visibility "Private"

New-MgGroup -DisplayName "M365-HR" `
  -MailNickname "M365-HR" `
  -GroupTypes @("Unified") `
  -MailEnabled $true `
  -SecurityEnabled $false `
  -Visibility "Private"

New-MgGroup -DisplayName "M365-AllCompany" `
  -MailNickname "M365-AllCompany" `
  -GroupTypes @("Unified") `
  -MailEnabled $true `
  -SecurityEnabled $false `
  -Visibility "Public"
```


## 🔐 4. Multi-Factor Authentication (MFA)

### Strategy
All users must register MFA before accessing any Microsoft 365 service. Conditional Access enforces MFA on all cloud apps for all users.

### Conditional Access Policy — Require MFA for All Users

| Setting | Value |
|---|---|
| Policy Name | CA-001: Require MFA — All Users |
| Users | All Users |
| Cloud Apps | All Cloud Apps |
| Conditions | Any location, any device |
| Grant Control | Require MFA |
| State | Enabled |

### Portal Steps
1. Navigate to **Entra ID > Security > Conditional Access**
2. Click **New Policy**
3. Name: `CA-001: Require MFA — All Users`
4. Assignments > Users: **All Users**
5. Target Resources: **All Cloud Apps**
6. Grant: **Require Multi-factor Authentication**
7. Enable policy: **On**
8. Click **Save**

### MFA Methods Configured
- Microsoft Authenticator (primary)
- SMS (fallback for users unable to install Authenticator)

> ⚠️ **Exception Process:** Users with older devices unable to install Microsoft Authenticator must submit a helpdesk ticket. SMS is enabled as a fallback and documented in the exception register.


## 🔑 5. Self-Service Password Reset (SSPR)

### Design

| Setting | Value |

| Scope | All Users |
| Authentication Methods Required | 2 |
| Methods Available | Authenticator App · Mobile Phone · Email |
| Registration Enforced | Yes — on next login |
| Reset Link Expiry | 30 minutes |

### Portal Steps
1. Navigate to **Entra ID > Users > Password Reset**
2. Properties: Enable for **All**
3. Authentication Methods: set to **2 required**
4. Enable: Mobile phone, Email, Microsoft Authenticator
5. Registration: **Require users to register on sign-in: Yes**
6. Click **Save**

> 💡 **Lesson Learned:** Always test the SSPR flow as a real end user before enabling for everyone. The admin view and the user experience are significantly different.



## 🚫 6. Custom Banned Passwords

### Banned Terms List
The following company-specific terms were added to the banned password list:

```
Greenwood
Accountants
GreenAcc
Finance2026
Accounts1
GwoodIT
Staff123
London2026
```

### Portal Steps
1. Navigate to **Entra ID > Security > Authentication Methods > Password Protection**
2. Enable **Custom Banned Passwords: Yes**
3. Enforce custom list: **Yes**
4. Add terms to the **Custom Banned Password List**
5. Click **Save**


## 🛡️ 7. Admin Role Assignments (Least Privilege)

### Role Assignment Register

| User | Role Assigned | Justification |
|---|---|---|
| Samuel Ejeh | User Administrator | Manage user accounts and group memberships |
| Emma Clarke | Exchange Administrator | Manage Exchange Online mailboxes |
| Ethan Cole | Teams Administrator | Manage Microsoft Teams configuration |

### PowerShell — Assign Roles
```powershell
# Get role IDs
Get-MgDirectoryRole | FL DisplayName, Id

# Assign User Administrator
New-MgDirectoryRoleMemberByRef `
  -DirectoryRoleId "<User-Administrator-Role-ID>" `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/<user-id>"
  }
```

> ⚠️ **Lesson Learned:** During early testing, Global Administrator was accidentally assigned instead of User Administrator. Always verify role scope before assignment. Least privilege must be enforced from day one.


## 📚 Runbooks

### Runbook 1 — Create New User

| Step | Action |

| 1 | Obtain approved user request form from HR |
| 2 | Connect to Microsoft Graph: `Connect-MgGraph -Scopes "User.ReadWrite.All"` |
| 3 | Create user with correct UPN format: `firstname.lastname@greenwoodaccountants.co.uk` |
| 4 | Set department, job title, and manager |
| 5 | Assign Microsoft 365 licence |
| 6 | Wait up to 30 minutes for Exchange mailbox provisioning |
| 7 | Add user to relevant security group(s) |
| 8 | Notify user to register MFA on first login |
| 9 | Confirm MFA registration within 24 hours |

### Runbook 2 — Assign Licence
```powershell
Set-MgUserLicense -UserId "<user-id>" `
  -AddLicenses @{SkuId = "<licence-sku-id>"} `
  -RemoveLicenses @()
```
> ⚠️ Exchange mailbox provisioning takes up to **30 minutes** after licence assignment. Document this on Day 1 onboarding to avoid unnecessary helpdesk tickets.

### Runbook 3 — Add User to Group
```powershell
New-MgGroupMember -GroupId "<group-id>" -DirectoryObjectId "<user-id>"
```
> ⚠️ Dynamic groups may take **5–30 minutes** to populate after rule evaluation. Always create dynamic groups before assigning policies.

### Runbook 4 — Reset MFA
1. Navigate to **Entra ID > Users > Select User > Authentication Methods**
2. Click **Require re-register MFA**
3. Notify the user to re-register on next login
4. Log the reset in the change record

### Runbook 5 — Disable Leaver Account
```powershell
# Disable account immediately
Update-MgUser -UserId "<user-id>" -AccountEnabled $false

# Revoke all active sessions
Revoke-MgUserSignInSession -UserId "<user-id>"

# Remove from all groups
Get-MgUserMemberOf -UserId "<user-id>" | ForEach-Object {
    Remove-MgGroupMemberByRef -GroupId $_.Id -DirectoryObjectId "<user-id>"
}

# Remove assigned licences
Set-MgUserLicense -UserId "<user-id>" `
  -AddLicenses @() `
  -RemoveLicenses @("<licence-sku-id>")
```

## 📋 Policy Register

### Conditional Access Policies

| Policy ID | Name | Users | Apps | Grant | Status |
|---|---|---|---|---|---|
| CA-001 | Require MFA — All Users | All Users | All Cloud Apps | Require MFA | Enabled |

### Group Register

| Group Name | Type | Members | Owner |
|---|---|---|---|
| SG-Finance-Users | Security (Assigned) | Emma greenwood, Daniel greenwood | Samuel greenwood |
| SG-HR-Users | Security (Assigned) | Fatima greenwood, Michael greenwood | Samuel greenwood |
| SG-IT-Admins | Security (Assigned) | Samuel greenwood, Ethan greenwood | Samuel greenwood |
| SG-Management | Security (Assigned) | Henry greenwood, Kemi greenwood| Samuel greenwood |
| SG-All-Staff | Security (Dynamic) | All enabled members | Samuel greenwood |
| M365-Finance | M365 Group | Finance staff | Emma greenwood |
| M365-HR | M365 Group | HR staff | Fatima greenwood |
| M365-AllCompany | M365 Group | All staff | Henry greenwood |

### Admin Role Assignments

| User | Role | Assigned By | Date |
|---|---|---|---|
| Samuel greenwood | User Administrator | Global Admin | 17/05/2026 |
| Emma greenwood | Exchange Administrator | Global Admin | 17/05/2026 |
| Ethan greenwood | Teams Administrator | Global Admin | 17/05/2026 |


## 🔧 Troubleshooting Guide

### Issue 1: Dynamic Group Not Populating
**Symptom:** New user not appearing in SG-All-Staff after creation.
**Cause:** Entra ID dynamic group rule evaluation delay (5–30 minutes).
**Fix:** Wait and re-check. Verify rule is correct:
```powershell
Get-MgGroup -GroupId "<group-id>" | FL MembershipRule, MembershipRuleProcessingState
```

### Issue 2: MFA Registration Failure
**Symptom:** User cannot complete MFA registration — Authenticator app not available on device.
**Fix:** Enable SMS as fallback in Authentication Methods. Log exception in the exception register. Update user's authentication method to Mobile Phone (SMS).

### Issue 3: SSPR Not Working for User
**Symptom:** User cannot reset password via SSPR portal.
**Cause:** User has not registered two authentication methods.
**Fix:**
1. Check registration status: **Entra ID > Users > Authentication Methods**
2. If not registered, require re-registration on next login
3. Confirm user has registered at least 2 methods

### Issue 4: Exchange Mailbox Not Provisioned
**Symptom:** New user cannot access email after account creation.
**Cause:** Exchange mailbox provisioning delay after licence assignment (up to 30 minutes).
**Fix:** Wait 30 minutes. Verify licence is assigned:
```powershell
Get-MgUserLicenseDetail -UserId "<user-id>" | FL SkuPartNumber
```

### Issue 5: UPN Conflict (Duplicate Names)
**Symptom:** Two users with identical first and last names — UPN conflict on creation.
**Fix:** Apply tie-breaker rule — append middle initial or department suffix:
- `john.a.greenwood@christtech.co.uk`
- `john.greenwood.finance@christtech.co.uk`
Document the tie-breaker rule in the naming convention before deployment.


## ⚡ Challenges Faced

| Challenge | Resolution |
|---|---|
| Dynamic group population delay | Created groups first, waited for population before assigning policies |
| MFA registration on older phones | Configured SMS as fallback, documented exception process |
| Global Admin assigned by mistake | Corrected to User Administrator; implemented role verification checklist |
| UPN conflict — duplicate names | Added tie-breaker rule to naming convention (middle initial or dept suffix) |
| Licence-to-mailbox delay | Added 30-minute wait note to onboarding runbook |

---

## 💡 Lessons Learned

1. **Always create dynamic groups before assigning policies** — the group must be populated before any policy assignment is meaningful.
2. **Least privilege is not theoretical** — using Global Administrator for routine tasks pollutes the audit log with over-privileged actions. Role discipline starts from day one.
3. **Naming conventions must cover edge cases before deployment** — two users with the same name exposed the gap immediately.
4. **Test SSPR as a real end user** — the admin view and the user experience are completely different.
5. **Document the licence-to-mailbox delay in the onboarding runbook** — it is the most common source of Day 1 helpdesk tickets.

---

## 📁 Repository Structure

```
Greenwood/
├── Project1-Identity-Foundation/
│   ├── README.md
│   ├── Design-Documents/
│   │   ├── Identity-Strategy.md
│   │   ├── User-Naming-Convention.md
│   │   ├── Group-Structure.md
│   │   ├── MFA-Strategy.md
│   │   └── SSPR-Design.md
│   ├── Policy-Register/
│   │   └── Policy-Register.md
│   ├── Runbooks/
│   │   ├── Create-New-User.md
│   │   ├── Assign-Licence.md
│   │   ├── Add-To-Group.md
│   │   ├── Reset-MFA.md
│   │   └── Disable-Leaver-Account.md
│   ├── Change-Record/
│   │   └── Change-Record-Identity-Foundation.md
│   └── Troubleshooting-Guide/
│       └── Troubleshooting-Guide.md
```

---

## 📝 Change Record

| Field | Details |
|---|---|
| **Change ID** | CHG-001 |
| **Title** | Identity Foundation Build — Microsoft Entra ID |
| **Date** | 17/05/2026 |
| **Implemented By** | Samuel Ejeh |
| **Approved By** | Global Administrator |
| **Description** | Created 10 test users, 5 security groups, 3 M365 groups, enabled MFA via Conditional Access, configured SSPR, set banned passwords, assigned least-privilege admin roles |
| **Status** | ✅ Completed |
| **Rollback Plan** | Disable Conditional Access policy CA-001, remove role assignments, delete test users and groups |

---

*Greenwood Accountants — Microsoft 365 Identity Foundation | Project 1 | May 2026*
