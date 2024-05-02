---
title: "Tip: Net Accounts Remediation Script"
last_modified_at: 2024-05-01T21:30:00
categories:
  - Quick Tips
tags:
  - Intune
  - PowerShell
---
Check out this quick Intune remediation script for optimizing Net Accounts settings. This sets the lockout threshold to 10 tries, lockout duration to 15 minutes and the reset lockout counter to 15 minutes.

Enhance local account security and boost your secure score, aligning with Center for Internet Security (CIS) benchmarks.

# Prerequisites
- Windows Enterprise


# Remediation Script

Let's dive into the detect and remediate scripts. Just copy-paste this and save as a Powershell script.

## NetAccounts-Detect.ps1

```powershell
<#
    .SYNOPSIS
        Checks wheter Lockout threshold is set at 10, reset lockout counter is set to 15 minutes and lockout duration is set at 15 minutes.
        Best practices from Microsoft for Secure Score.

    .NOTES
        Author: Diégo Derksen
        linkedIn: www.linkedin.com/in/diego-derksen
#>

$secpol = secedit /export /cfg secpol.cfg

$lockoutThreshold = (Get-Content secpol.cfg | Select-String "LockoutBadCount").ToString().Split('=')[1].Trim()
$resetLockoutCounter = (Get-Content secpol.cfg | Select-String "ResetLockoutCount").ToString().Split('=')[1].Trim()
$lockoutDurationValue = (Get-Content secpol.cfg | Select-String "LockoutDuration").ToString().Split('=')[1].Trim()

if ($lockoutThreshold -eq "10" -and $resetLockoutCounter -eq "15" -and $lockoutDurationValue -eq "15") {
    Write-Host "Values are correct"
    Remove-Item secpol.cfg
    exit 0
} else {
    Write-Host "Values are not correct"
    Remove-Item secpol.cfg
    exit 1
}
```

## NetAccounts-Remediate.ps1

```powershell
<#
    .SYNOPSIS
        Sets Lockout threshold to 10, reset lockout counter to 15 minutes and lockout duration to 15 minutes.
        Best practices from Microsoft for Secure Score.

    .NOTES
        Author: Diégo Derksen
        linkedIn: www.linkedin.com/in/diego-derksen
#>

$threshold = 10
$resetLockoutCounterAfter = 15
$lockoutDuration = 15

$secpol = secedit /export /cfg secpol.cfg

$lockoutThreshold = (Get-Content secpol.cfg | Select-String "LockoutBadCount").ToString().Split('=')[1].Trim()
$resetLockoutCounter = (Get-Content secpol.cfg | Select-String "ResetLockoutCount").ToString().Split('=')[1].Trim()
$lockoutDurationValue = (Get-Content secpol.cfg | Select-String "LockoutDuration").ToString().Split('=')[1].Trim()

if ($lockoutThreshold -ne $threshold -or $resetLockoutCounter -ne $resetLockoutCounterAfter -or $lockoutDurationValue -ne $lockoutDuration) {
    (Get-Content secpol.cfg).Replace("LockoutBadCount = $lockoutThreshold", "LockoutBadCount = $threshold").Replace("ResetLockoutCount = $resetLockoutCounter", "ResetLockoutCount = $resetLockoutCounterAfter").Replace("LockoutDuration = $lockoutDurationValue", "LockoutDuration = $lockoutDuration") | Set-Content secpol.cfg
    secedit /configure /db secedit.sdb /cfg secpol.cfg /areas SECURITYPOLICY
    Remove-Item secpol.cfg
    Remove-Item C:\Windows\security\logs\scesrv.log
    Write-Host "Values have been changed"
    exit 0
} else {
    Remove-Item secpol.cfg
    Write-Host "Values haven't been changed"
    exit 1
}
```

# Import into Intune

1. Go to https://intune.microsoft.com
1. Go to Devices
1. Go to Scripts and Remediations
1. Create a remediation script
1. Pick a name for the script
1. Import the detection and remediation script
    - Run this script using the logged-on credentials: **No**
    - Enforce script signature check: **No**
    - Run script in 64-bit Powershell: **Yes**

