---
title: "Personal OneDrive accounts check for Roadmap ID 490064"
last_modified_at: 2025-04-24T13:33:00+02:00
categories:
  - Blog
tags:
  - Intune
---

Microsoft has recently introduced a helpful (sigh) new feature ([Roadmap ID 490064](https://www.microsoft.com/nl-nl/microsoft-365/roadmap?id=490064)) that notifies end users if they want to synchronize their personal documents on a managed device. Before enforcing the blocking policy, it's important to assess the potential impact. If we find that only a small number of users are affected, implementation will be easier to manage and communicate. However, if a significant number of users are using personal OneDrive accounts, you can broaden your communication.


**What is my goal?**  
Check how many users currently have personal OneDrive accounts configured on their device, using a PowerShell script deployed via Intune as a remediation script.


Below is the PowerShell script we use to detect personal accounts. It checks the local registry for any OneDrive accounts and filters out allowed corporate domains for example contoso.com

```powershell
# Define allowed corporate domains
$allowedDomains = @("contoso.com")

# Initialize values
$personalFound = $false
$domains = @()

# Base OneDrive account registry key
$baseKey = "HKCU:\Software\Microsoft\OneDrive\Accounts"

if (Test-Path $baseKey) {
    Get-ChildItem -Path $baseKey | ForEach-Object {
        $subKeyPath = $_.PSPath

        try {
            $keyProps = Get-ItemProperty -Path $subKeyPath -ErrorAction Stop
            $email = $keyProps.UserEmail

            if ($email) {
                $domain = $email.Split("@")[-1].ToLower()
                if ($allowedDomains -notcontains $domain) {
                    $personalFound = $true
                    if ($domains -notcontains $domain) {
                        $domains += $domain
                    }
                }
            }
        } catch {
            # Skip unreadable keys
        }
    }
}

# Output simplified result
$summary = @{
    PersonalOneDriveFound = $personalFound
    Domains = $domains
}

$summary | ConvertTo-Json -Compress

# Set exit code
if ($personalFound) {
    exit 1
} else {
    exit 0
}
```

Save this script with a .ps1 extension. You can then upload and deploy it in Intune as a remediation script.

**Intune Settings**  
When uploading the script in Intune, use the following settings:

Run this script using the logged-on credentials: Yes

Enforce script signature check: No

Run script in 64-bit PowerShell: Yes

The script exits with code 1 if any personal accounts are found, which allows for easy detection and reporting across your fleet. The output includes which non-corporate domains were discovered.

**Next Steps**  
Once we gather enough data on current usage, we can make an informed decision on when and how to enforce the DisablePersonalSync (GPO Name) or as it's called in Intune: "Prevent users from syncing personal OneDrive accounts". The goal is to improve security while minimizing disruption to users.
