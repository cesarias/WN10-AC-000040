# WN10-AC-000040
# Windows 10 STIG Remediation: WN10-AC-000040

## 🔐 STIG Overview

**STIG ID:** WN10-AC-000040
**Title:** The built-in Microsoft password complexity filter must be enabled.
**Category:** Account Policy
**Severity:** Medium
**Reference:** [DISA STIG](https://public.cyber.mil/stigs/)

## 🌟 Objective

Remediate the finding `WN10-AC-000040` by enabling the Microsoft password complexity filter. Demonstrate manual and PowerShell-based remediation, validate with Tenable scans, and document each step using an Azure-hosted Windows 10 VM.

---

## 📊 Technology Stack

* **Scanner:** Tenable (Nessus/Tenable.sc with STIG plugin enabled)
* **Platform:** Azure Windows 10 VM
* **Tools:** Local Security Policy (secpol.msc), PowerShell

---

## 📖 Steps Performed

### 1. Initial Scan

* **Tool Used:** Tenable
* **Result:** Finding `WN10-AC-000040` marked as **non-compliant**
<img width="1000" alt="image" src="https://i.imgur.com/gGAI51n.png">
---

### 2. Manual Remediation

* **Steps:**

  1. Press `Win + R`, type `secpol.msc`, press Enter
  2. Navigate to `Account Policies > Password Policy`
  3. Set **Password must meet complexity requirements** to **Enabled**
* **Screenshot of GUI change:** `images/manual-remediation-setting.png`
* **Post-remediation scan result:** Compliant
<img width="1000" alt="image" src="https://i.imgur.com/UyrxbY9.png">
<img width="1000" alt="image" src="https://i.imgur.com/4B8Dcli.png">

---

### 3. Undo Manual Fix (Validation Step)

* Set policy back to **Disabled** or **Not Defined**
* Rescan to confirm the finding returns
<img width="1000" alt="image" src="">

---

### 4. PowerShell Remediation

```powershell
# Run this script in an elevated PowerShell window
secedit /export /cfg C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("PasswordComplexity = 0", "PasswordComplexity = 1") | Set-Content C:\secpol.cfg
secedit /configure /db secedit.sdb /cfg C:\secpol.cfg /areas SECURITYPOLICY
Remove-Item C:\secpol.cfg -Force
gpupdate /force
```

* **Script Name:** `remediate-WN10-AC-000040.ps1`
* **Screenshot of script execution:** `images/powershell-script-run.png`
* **Rescan Result:** Compliant
<img width="1000" alt="image" src="https://i.imgur.com/iFYAiny.png">

---

## 📃 Registry/Policy Details

* **Setting Location (GUI):** Local Security Policy > Password Policy
* **Registry Path (for auditing):**

  * `HKLM\SYSTEM\CurrentControlSet\Control\Lsa` (used by complexity filter)
* **Expected Value:** Password complexity requirement = Enabled

---

## 🔄 Before & After Comparison

| State  | Screenshot                     |
| ------ | ------------------------------ |
| Before | `images/initial-scan-fail.png` |
| After  | `images/post-script-pass.png`  |

---

## 📓 Lessons Learned

* Complexity filters are key to protecting from password spraying and dictionary attacks
* PowerShell can fully replicate GUI security changes and scale across environments
* Tenable provides effective validation for STIG remediation in real time

---

## 📂 Files Included

* `remediate-WN10-AC-000040.ps1` – PowerShell remediation script
* `manual-steps.txt` – Written remediation procedure
* `images/` – Screenshots folder (initial, manual fix, script fix, failure states)

---

## 👤 Author

**Name:** Cesar Arias
**Role:** Cybersecurity Student | STIG Lab Project
**Contact:** cesarias2022@gmail.com

---

## 🗓️ Date Completed

**Remediation Date:** 7/11/2025

---

# Password Policy Implementation Lab

## 🔐 Overview

This section demonstrates how to implement a realistic, enterprise-grade password policy on a Windows 10 system. It includes both GUI-based and PowerShell-based methods to configure security settings and force a password change for the current user.

---

## 🌐 Enterprise Password Policy

| Setting                          | Value                            |
| -------------------------------- | -------------------------------- |
| Minimum Password Length          | 14 characters                    |
| Maximum Password Age             | 60 days                          |
| Minimum Password Age             | 1 day                            |
| Enforce Password History         | 24 previous passwords remembered |
| Password Complexity Requirements | Enabled                          |
| Account Lockout Threshold        | 5 invalid attempts               |
| Account Lockout Duration         | 15 minutes                       |
| Reset Lockout Counter After      | 15 minutes                       |

---

## 🛠️ Manual Implementation (Local Security Policy GUI)

1. **Open Local Security Policy**

   * Press `Win + R`, type `secpol.msc`, and press Enter

2. **Configure Password Policy**

   * Navigate to: `Account Policies > Password Policy`
   * Apply the following settings:

     * Enforce password history: **24**
     * Maximum password age: **60**
     * Minimum password age: **1**
     * Minimum password length: **14**
     * Password must meet complexity requirements: **Enabled**

3. **Configure Account Lockout Policy**

   * Navigate to: `Account Policies > Account Lockout Policy`
   * Apply the following settings:

     * Account lockout threshold: **5**
     * Account lockout duration: **15 minutes**
     * Reset account lockout counter after: **15 minutes**

---

## 💻 PowerShell Script Implementation

```powershell
# Export current security settings
secedit /export /cfg C:\secpol.cfg

# Modify settings
(Get-Content C:\secpol.cfg).replace("PasswordHistorySize = 0", "PasswordHistorySize = 24") | Set-Content C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("MaximumPasswordAge = 42", "MaximumPasswordAge = 60") | Set-Content C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("MinimumPasswordAge = 0", "MinimumPasswordAge = 1") | Set-Content C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("MinimumPasswordLength = 0", "MinimumPasswordLength = 14") | Set-Content C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("PasswordComplexity = 0", "PasswordComplexity = 1") | Set-Content C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("LockoutBadCount = 0", "LockoutBadCount = 5") | Set-Content C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("LockoutDuration = 30", "LockoutDuration = 15") | Set-Content C:\secpol.cfg
(Get-Content C:\secpol.cfg).replace("ResetLockoutCount = 30", "ResetLockoutCount = 15") | Set-Content C:\secpol.cfg

# Apply updated policy
secedit /configure /db secedit.sdb /cfg C:\secpol.cfg /areas SECURITYPOLICY

# Clean up
Remove-Item C:\secpol.cfg -Force

# Force update and confirm
gpupdate /force
net accounts
```

---

## 🚀 Force Password Change on Next Logon

### For current user:

```powershell
wmic useraccount where name='%username%' set PasswordExpires=True
```

### For specific user:

```powershell
net user YourUserName NewSecurePassword1!
wmic useraccount where name='YourUserName' set PasswordExpires=True
```

---

## 📆 Notes

* Use **Tenable/Nessus** to verify that the STIG `WN10-AC-000040` is marked compliant.
* Document before/after scans to demonstrate successful remediation.
* Pair this with a simulated password change event to show compliance in practice.


## 📄 Version

**Version:** 1.0
**Environment:** Azure Windows 10 VM
**Audit Tool:** Tenable (STIG Audit Plugins)
