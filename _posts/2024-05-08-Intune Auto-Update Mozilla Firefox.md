---
title: "Post: Intune Auto-Update Mozilla Firefox"
last_modified_at: 2024-05-08T18:10:00
categories:
  - Blog
tags:
  - Intune
---
In this post, I’ll walk you through the process of setting up automatic updates for Mozilla Firefox using Intune. As an Intune administrator, it’s crucial to ensure that all enrolled devices remain secure. Managing Firefox updates effectively is essential to maintain security, stability, and optimal performance.

This will be a series on enabling automatic updates for various web browsers. [Check out the last post regarding automatically updating Google Chrome](https://diegoderksen.github.io/blog/Intune-Auto-Update-Google-Chrome/)
# Prerequisite:

- [Firefox ADMX Templates](https://github.com/mozilla/policy-templates/releases)
# Steps
## Firefox ADMX templates

1. Download "Policy_Templates.zip"
1. Extract the zip file to a easy findable place

## Importing ADMX into intune

1. Go to the Intune dashboard
2. Go to "Devices"
3. Go to "Configuration"
4. On the "Configuration" page you will find "Import ADMX" at the top of the page, click on this button
5. Click on "Import"

## Mozilla ADMX Import:

1. Click on "select an .admx file" and find the mozilla.admx inside the following folder: `policy_templates_v5.10\windows` of the extracted Mozilla Firefox ADMX .zip file
1. Click on "Select an .adml file" and select the language folder you want (In my case en-US) and find mozilla.adml in this folder

    It should look like this:

    ![MozillaADMX](/assets/images/Intune-Auto-Update-Browsers/Firefox/Mozilla%20ADMX%20import.png)
  

## Firefox ADMX Import:

1. Click on Import once again
1. Click on "select an .admx file" and find the firefox.admx inside the following folder: `policy_templates_v5.10\windows` of the extracted Mozilla Firefox ADMX .zip file
1. Click on "Select an .adml file" and select the language folder you want (In my case en-US) and find firefox.adml in this folder.

    It should look like this:

    ![FirefoxADMX](/assets/images/Intune-Auto-Update-Browsers/Firefox/Firefox%20ADMX%20import.png)

## Device configuration:

1. Go to Configuration profiles and create a new policy
1. By platform choose "Windows 10 and later" and profile type "Templates" followed by template name: "Imported Administrative templates (Preview)"
1. Pick a name for the policy
1. on the Configuration Settings tab, search for:

   - "**Application Autoupdate**" and set this to enabled  

   - "**Background updater**" and set this to enabled  

   - "**Extension Update**" and set this to enabled  

1. Choose a scope tag if you use them
1. Assign the policy to users and create the policy.

You have now set the policy inside Intune to auto update Mozilla Firefox. Even if the user hasn't opened the application.

  

## How do I know if the policy has hit the device?

You can type in `about:policies` in the URL bar inside Firefox to check whether the policy has hit the device.