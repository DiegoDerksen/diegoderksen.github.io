---
title: "Tip: Windows Pro to Enterprise Remediate Script"
last_modified_at: 2024-05-04T18:00:00
categories:
  - Quick Tips
tags:
  - Intune
  - PowerShell
---

Thanks to [Rudy Ooms his blogpost](https://call4cloud.nl/2024/05/kb5036980-breaks-upgrade-windows11-enterprise/) there is a fix (or 2) for Windows 11 not upgrading to Enterprise while having a E5 license that has been an issue since the latest Windows 11 Update KB5036980 and build 22631.3447.
This is because ClipRenew does not make mfarequiredcliprenew in the registry and not having the right permissions to make this key. 

I have made a remediation script to keep track on which devices are pro or enterprise. If it's pro it automatically starts the script from Rudy and upgrades to Windows 11 Enterprise.

Microsoft's product team is already alerted of this issue and it will be fixed in the future.

# Remediation Script

Let's dive into the detect and remediate scripts. Just copy-paste this and save as a Powershell script.

## Windows-Enterprise-or-pro-Detect.ps1

```powershell
# Define registry key path and values
$registryPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\MfaRequiredInClipRenew"
$registryValueName = "Verify Multifactor Authentication in ClipRenew"
$expectedRegistryValueData = 0

if (Test-Path -Path $registryPath) {
    $currentRegistryValue = (Get-ItemProperty -Path $registryPath -Name $registryValueName).$registryValueName
    if ($currentRegistryValue -eq $expectedRegistryValueData) {
        Write-Output "Remediation successful: Registry key exists and value matches expected value."
        Exit 0
    } else {
        Write-Output "Remediation failed: Registry key exists but value does not match expected value."
        Exit 1
    }
} else {
    Write-Output "Remediation failed: Registry key does not exist."
    Exit 1
}
```

## Windows-Enterprise-Remediate.ps1

```powershell
# Define registry key path and values
$registryPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\MfaRequiredInClipRenew"
$registryValueName = "Verify Multifactor Authentication in ClipRenew"
$registryValueData = 0  # DWORD value
$sid = New-Object System.Security.Principal.SecurityIdentifier("S-1-1-0")  # Everyone group SID

# Check if the registry key already exists
if (-not (Test-Path -Path $registryPath)) {
  # If the key doesn't exist, create it and set the DWORD value
  New-Item -Path $registryPath -Force | Out-Null
  Set-ItemProperty -Path $registryPath -Name $registryValueName -Value $registryValueData -Type DWORD
  Write-Output "Registry key created and DWORD value added."
} else {
  Write-Output "Registry key already exists. No changes made."
}

# Add read permissions for "Everyone" group
Write-Output "Granting read permissions..."
$acl = Get-Acl -Path $registryPath
$ruleSID = New-Object System.Security.AccessControl.RegistryAccessRule($sid, "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.AddAccessRule($ruleSID)
Set-Acl -Path $registryPath -AclObject $acl

Start-Process "$env:SystemRoot\system32\ClipRenew.exe"
```

# Create a filter
1. Go to https://intune.microsoft.com
2. Go to Tentnat Administration
3. Go to filters
4. Create a new filter with the following syntax:

```
(device.osVersion -startsWith "10.0.22631.3447") and (device.deviceOwnership -eq "Corporate") and (device.operatingSystemSKU -in ["Professional"])
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
1. Be sure to use a device group and apply the filter from above as Include
1. Schedule should be set to **Once**

New devices will be immediately upgraded to Enterprise. Current pro devices will be upgraded to Enterprise after a restart.
