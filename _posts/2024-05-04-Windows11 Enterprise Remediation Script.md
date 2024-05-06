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

# Remediation Script

Let's dive into the detect and remediate scripts. Just copy-paste this and save as a Powershell script.

## Windows-Enterprise-or-pro-Detect.ps1

```powershell
# Define the registry key path and value
$registryPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\MfaRequiredInClipRenew"
$registryValueName = "Verify Multifactor Authentication in ClipRenew"

# Check if the registry key already exists
if (Test-Path -Path $registryPath) {
    # If the key exists, check the value
    $value = Get-ItemProperty -Path $registryPath -Name $registryValueName
    if ($value -eq 0) {
        Write-Output "Registry key and value exist. No changes needed."
               exit 0 # Success
    } else {
        Write-Output "Registry key exists but value is incorrect."
                exit 1 # Configuration found but not compliant
    }
} else {
    Write-Output "Registry key does not exist."
            exit 1 # Does not exist
}
```

## Windows-Enterprise-Remediate.ps1

```powershell
# Define registry key path and values
$registryPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\MfaRequiredInClipRenew"
$registryValueName = "Verify Multifactor Authentication in ClipRenew"
$registryValueData = 0  # DWORD value
$sid = New-Object System.Security.Principal.SecurityIdentifier("S-1-1-0")  # Everyone group SID

# Check if registry key exists
if (-not (Test-Path -Path $registryPath)) {
  Write-Output "Creating missing registry key..."
  New-Item -Path $registryPath -Force | Out-Null
}

# Set registry value
Write-Output "Setting registry value..."
Set-ItemProperty -Path $registryPath -Name $registryValueName -Value $registryValueData -Type DWORD

# Add read permissions for "Everyone" group
Write-Output "Granting read permissions..."
$acl = Get-Acl -Path $registryPath
$ruleSID = New-Object System.Security.AccessControl.RegistryAccessRule($sid, "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.AddAccessRule($ruleSID)
Set-Acl -Path $registryPath -AclObject $acl

Start-Process "$env:SystemRoot\system32\ClipRenew.exe"

Write-Output "Remediation complete."
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
