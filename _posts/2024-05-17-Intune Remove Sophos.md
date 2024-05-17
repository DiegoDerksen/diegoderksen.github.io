---
title: "Post: Remove Sophos Endpoint Central with Intune"
last_modified_at: 2024-05-017T11:00:00
categories:
  - Blog
tags:
  - Intune
---

Managing security software across an organization's fleet of devices can be a complex task, especially when transitioning between different solutions. If you're using Microsoft Intune to manage your endpoints and have decided to remove Sophos antivirus software, you've come to the right place. This guide will walk you through the steps to efficiently and effectively uninstall Sophos from devices managed by Intune, ensuring a smooth transition and maintaining security compliance throughout your network. Whether you're switching to another antivirus solution or restructuring your cybersecurity strategy, this tutorial will help you navigate the process with ease.

# Prerequisites
- [Win32 Content Prep Tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool)

# Steps

## Turn off Tamper Protection
1. In Sophos Central, click My Products > General Settings.
1. Under General, click  Tamper Protection.
1. Move the slider to the left to turn off tamper protection, then click Save. 
## Create a batch file

1. Open Notepad
1. Copy and paste the following command: `"C:\Program Files\Sophos\Sophos Endpoint Agent\SophosUninstall.exe" --quiet`
1. Create a new folder and save as "Uninstall_Sophos.bat"

## Package
1. Open the Win32 Content Prep Tool
1. copy and paste the folder path to the CMD window as "Source folder"
1. Drag the .bat file as "Setup file"
1. Specify the output of the .intunewin file
1. Don't create a catalog folder

## Importing into Intune
1. Go to the Intune Portal
1. Go to Apps > Windows
1. Click on Add > Windows app (Win32)
1. Add the created .intunewin file and enter the information
1. In step enter the following install/uninstall command: `Uninstall_Sophos.bat`
1. You can choose the requirements
1. Detection rule should be: 
    - Rule type: File
   - Path: %ProgramFiles%\Sophos\Sophos UI
   - File or folder: Sophos UI.exe
   - Detection method: File or folder exists.
1. Choose the Uninstall assignment and assign to users or devices.  

Always test on a test device.

# Conclusion

Transitioning your organization's security software is a significant undertaking, but with the right tools and a clear process, it can be managed efficiently. By following this guide, you have learned how to remove Sophos antivirus software from devices managed by Microsoft Intune. This step-by-step approach ensures that your devices remain compliant and secure throughout the transition period. Always remember to test the process on a test device before deploying it widely to avoid any disruptions. Whether you're adopting a new antivirus solution or simply changing your cybersecurity strategy, this guide provides a solid foundation to ensure a smooth and effective migration. By leveraging Intune’s capabilities, you can maintain a secure and well-managed network environment.