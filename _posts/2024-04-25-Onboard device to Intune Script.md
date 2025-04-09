---
title: " Onboard devices to Intune with script"
last_modified_at: 2024-04-25T11:00:00
categories:
  - Quick Tips
tags:
  - Intune
---

In this edition of quick tips, we're delving into the smooth journey of onboarding your device to Intune via the Hardware Hash, all powered by a convenient PowerShell script.

----------------

1. Access the Command Prompt by pressing Shift + F10 during the language selection step of a new Windows installation.

	**Info Notice:** Ensure that your device is connected to the internet before proceeding.
	{: .notice--info}

2. Type the following command in the cmd prompt to open Powershell

	```powershell
	 Powershell
	```

3. Execute the following command in PowerShell to set the execution policy for the current process to unrestricted:

	```powershell
	Set-ExecutionPolicy -Scope Process -ExecutionPolicy Unrestricted
	```

4. Install the script named “Get-WindowsAutoPilotInfo” using the following command:

	```powershell
	Install-Script -Name Get-WindowsAutoPilotInfo
	```

5. After the script has been installed, type in the following script:

	```powershell
	Get-WindowsAutoPilotInfo.ps1 -Online
	```

6. Login with Microsoft credentials from a administrator

--------------


Your device has successfully been onboarded to Intune with the Hardware Hash. Typically, it may take between 5 to 10 minutes for a profile to be assigned to the device. Once completed, you'll be able to log in using a user account associated with your tenant.
