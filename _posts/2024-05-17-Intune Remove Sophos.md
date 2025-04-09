---
title: "Remove Sophos Endpoint with Intune"
last_modified_at: 2024-05-17T11:00:00
categories:
  - Blog
tags:
  - Intune
---

I'll guide you through removing Sophos antivirus from computers managed by Microsoft Intune. I'll show you each step to safely uninstall Sophos while keeping your devices secure.

# **Prerequisites**

- [Win32 Content Prep Tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool)

# **Steps**

## **Turn off Tamper Protection**

1. Open Sophos Central and click My Products > General Settings
2. Find and click Tamper Protection under General
3. Turn off the protection by sliding the switch left, then save your changes

## **Create a batch file**

1. Open Notepad on your computer
2. Copy this command: `"C:\Program Files\Sophos\Sophos Endpoint Agent\SophosUninstall.exe" --quiet`
3. Make a new folder and save the file as `Uninstall_Sophos.bat`

## **Create the package**

1. Start the Win32 Content Prep Tool
2. In the command window, enter your folder location as the "Source folder"
3. Add your .bat file as the "Setup file"
4. Choose where to save the .intunewin file
5. Skip creating a catalog folder

## **Add to Intune**

1. Log into the Intune Portal
2. Go to Apps > Windows
3. Select Add > Windows app (Win32)
4. Upload your .intunewin file and fill in the required information
5. Use this command for installation/uninstallation: `Uninstall_Sophos.bat`
6. Set up any requirements you need
7. For detection rules, use these settings:
    - Rule type: File
    - Path: %ProgramFiles%\Sophos\Sophos UI
    - File or folder: Sophos UI.exe
    - Detection method: File or folder exists.
8. Set up the uninstall assignment and choose which users or devices should get it

🚨Always test this process on one device first before using it on all your devices.
{: .notice--danger}


# **Conclusion**

Follow these steps carefully to remove Sophos from your devices. Remember to test everything on one device first to avoid any problems across your network.
