---
title: "Post: Microsoft Teams Remediation Script"
last_modified_at: 2024-04-24T12:00:00
categories:
  - Blog
tags:
  - Intune
  - PowerShell
---

Are your users seeing a firewall Window regarding Teams when they first logon? Fear not! In this article, I'll walk you through a swift solution using an Intune remediation script.


This nifty script performs a simple yet effective task: it checks whether a teams.exe firewall rule exists. If not, it promptly creates a rule based on the userprofile Teams path.

# Prerequisites
- Windows Enterprise


# Remediation Script

Let's dive into the detect and remediate scripts. Just copy-paste this and save as a Powershell script.

## Teams-Detect.ps1

```powershell
<#
    .SYNOPSIS
        Detect whether the Microsoft Teams firewall policies have been created

    .NOTES
        Author: Diégo Derksen
        linkedIn: www.linkedin.com/in/diego-derksen
#>

# Define the name of the firewall rule
$firewallRuleName = "teams.exe"

# Get the firewall rule
$firewallRule = Get-NetFirewallRule -DisplayName $firewallRuleName -ErrorAction SilentlyContinue

# Check if the firewall rule exists
if ($null -ne $firewallRule) {
    # Firewall rule exists
    Write-Host "Teams Rules OK"
    exit 0
} else {
    # Firewall rule does not exist
    Write-Warning "Teams rules not OK"
    exit 1
}
```

## Teams-Remediate.ps1

```powershell
<#
    .SYNOPSIS
Create Teams firewall rules

    .NOTES
        Author: Diégo Derksen
        linkedIn: www.linkedin.com/in/diego-derksen
#>

Function Get-LoggedInUserProfile() {
# Getting user profile for Teams localdata path

    try {
    
       $loggedInUser = Gwmi -Class Win32_ComputerSystem | select username -ExpandProperty username
       $username = ($loggedInUser -split "\\")[1]

       $userProfile = Get-ChildItem (Join-Path -Path $env:SystemDrive -ChildPath 'Users') | Where-Object Name -Like "$username*" | select -First 1
       
    } catch [Exception] {
    
       Write-error "Unable to find logged in users profile folder. User is not logged on to the primary session: $_"
       Exit 1
    }

    return $userProfile
}

Function Set-TeamsFWRule($ProfileObj) {
# Setting up the inbound firewall rule
    $progPath = Join-Path -Path $ProfileObj.FullName -ChildPath "AppData\Local\Microsoft\Teams\Current\Teams.exe"

    if ((Test-Path $progPath)) {
            $ruleName = "Teams.exe"
            New-NetFirewallRule -DisplayName "$ruleName" -Direction Inbound -Profile Domain -Program $progPath -Action Allow -Protocol Any
            New-NetFirewallRule -DisplayName "$ruleName" -Direction Inbound -Profile Public,Private -Program $progPath -Action Block -Protocol Any

        }             
            
        }
Try {
    #Combining the two function in order to set the Teams Firewall rule for the logged in user
    Set-TeamsFWRule -ProfileObj (Get-LoggedInUserProfile)
    Write-Host "Teams rules OK."
Exit 0

} catch [Exception] {
    
 # Rules not made
    Write-Error "Teams Rules not OK"
   Exit 1
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

Et voila, your users won't see the firewall prompt for Teams when they first startup Teams.