---
title: "Post: Intune Auto-Update Google Chrome"
last_modified_at: 2024-04-23T16:00:00
categories:
  - Blog
tags:
  - Web browser
  - Intune
---

In this post, I’ll walk you through the process of setting up automatic updates for Google Chrome using Intune. As an Intune administrator, it’s crucial to ensure that all enrolled devices remain secure. Managing Chrome updates effectively is essential to maintain security, stability, and optimal performance.

This will be a series on enabling automatic updates for various web browsers.

# Prerequisite:
- Windows ADMX template [W10 22h2](https://www.microsoft.com/en-us/download/details.aspx?id=104677 "W10 22h2") -  [W11 22h2](https://www.microsoft.com/en-ie/download/details.aspx?id=105093 "W11 22h2")
- [Google Chrome ADM-/ADMX-templates & Google Updater ADMX-template](https://chromeenterprise.google/browser/download/#manage-policies-tab "Google Chrome adm-/admx-templates & Google Updater ADMX-template")

# Steps
We need to import both Google/Chrome and Windows ADMX files into Intune. That's because Chrome still requires the Windows ADMX template. 

## Windows ADMX templates
1. Download the Windows ADMX template for the right operating system version
1. Run the .msi package to extract the templates
1. The packages will be extracted to: `C:\Program Files (x86)\Microsoft Group Policy\`

## Google Chrome ADMX templates
1. Download both "Chrome ADM/ADMX templates" and "Google updater ADMX template update"
1. Extract both zip files to a easy findable place

## Importing ADMX into intune
1. Go to the Intune dashboard
1. Go to "Configuration"
1. On the "Configuration" page you will find "Import ADMX" at the top of the page, click on this button
1. Click on "Import"

    **Info Notice:** We first need to import the Windows ADMX before importing the Google.admx template.
    {: .notice--info}

### Windows ADMX Import:

1. Click on "select an .admx file" and find the windows.admx inside the following folder: `\PolicyDefinitions` of the extracted .msi Windows admx template file
1. Click on "Select an .adml file" and select the language folder you want (In my case en-US) and find Windows.adml in this folder

    It should look like this:
     ![WindowsADMX](/assets/images/Intune-Auto-Update-Browsers/Chrome/Windows%20admx%20import.png)
	
### Google ADMX Import:
1. Click on Import once again 
1. Click on "select an .admx file" and find the google.admx inside the following folder: `policy_templates\windows\admx` of the extracted Google ADMX .zip file
1. Click on "Select an .adml file" and select the language folder you want (In my case en-US) and find google.adml in this folder

    It should look like this:
    ![GoogleADMX](/assets/images/Intune-Auto-Update-Browsers/Chrome/Google%20ADMX%20import.png)
	
### Chrome ADMX Import:
1. Click on Import once again 
1. Click on "select an .admx file" and find the chrome.admx inside the following folder: `policy_templates\windows\admx` of the extracted Google ADMX .zip file
1. Click on "Select an .adml file" and select the language folder you want (In my case en-US) and find chrome.adml in this folder

    It should look like this:
    ![ChromeADMX](/assets/images/Intune-Auto-Update-Browsers/Chrome/Chrome%20ADMX%20import.png)
	
### Google Updater ADMX Import:
1. Click on Import once again 
1. Click on "select an .admx file" and find the GoogleUpdate.admx inside the following folder: `googleupdateadmx\GoogleUpdateAdmx` of the extracted Google Updater ADMX .zip file
1. Click on "Select an .adml file" and select the language folder you want (In my case en-US) and find GoogleUpdate.adml in this folder

    It should look like this:
    ![GoogleUpdateADMX](/assets/images/Intune-Auto-Update-Browsers/Chrome/Google%20Update%20ADMX%20import.png)
	
## Device configuration:
1. Go to Configuration profiles and create a new policy
1. By platform choose "Windows 10 and later" and profile type "Templates" followed by template name: "Imported Administrative templates (Preview)"
1. Pick a name for the policy
1. on the Configuration Settings tab, search for: 
   - "**Auto-update check period override**" and set this to enabled and 120 minutes
   - "**Enable component updates in Google Chrome**" and set this to enabled, 
   - "**Update policy override default**" and set this to enabled and "Always allow updates (recommended)"
1. Choose a scope tag if you use them
1. Assign the policy to users and create the policy.

You have now set the policy inside Intune to auto update Chrome.

## How do I know if the policy has hit the device?

You can type in `chrome://policy` in the URL bar inside Chrome to check whether the policy has hit the device.