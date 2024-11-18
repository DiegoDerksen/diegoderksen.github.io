---
title: "Post: Microsoft Defender for Android Low Touch Onboarding"
last_modified_at: 2024-11-18T20:00:00+02:00
categories:
  - Blog
tags:
  - Microsoft Defender
---

Microsoft has introduced Low Touch Onboarding in preview for Intune, designed to simplify device setup and reduce manual IT intervention. This feature automates much of the enrollment process, saving time and effort while improving the end-user experience.

In this quick guide, I'll show you how to set up Low Touch Onboarding in Intune.

# Defender on Managed Google Play
We first need to download Microsoft Defender in the Managed Google Play store.

1. Go to Apps inside Intune
2. Go to Android
3. Click on "Add" and select "Managed Google Play app"
4. Search up "Microsoft Defender: Antivirus" 
5. Click on "Microsoft Defender: Antivirus" and click on "Select"

We now have Defender inside our Intune tenant. We should push this when we setup other settings to streaming the user experience for end users.

# Setting up Always-on VPN
Let's do some configuration. We first need to setup the Always-On VPN for Defender. Defender for  Android uses a VPN in order to provide the Web Protection feature. This VPN is not a regular VPN. Instead, it's a local/self-looping VPN that does not take traffic outside the device.

1. Go to the Intune portal
2. Go to Android configuration policies
3. Create a new policy
4. Navigate to "Connectivity"
5. Enable "Always-on VPN (work profile lever)"
6. Set "VPN Client" to Custom
7. Set the "Package ID" to com.microsoft.scmx
8. Push this config to your users and save the profile

This step ensures that the user does not have to do the VPN install step.

# App Configuration policies
In the second step we need to configure a app configuration policy. This ensures that features are turned on and that Low Touch Onboarding is turned on.

1. Go to Apps inside Intune
2. Go to App Configuration policies
3. Click on "Create" and then "Managed devices"
4. Fill in the name and platform should be "Android Enterprise", Profile type should be "Fully Managed, Dedicated and Corporate-Owned Work Profile Only" and the targeted app should be "Microsoft Defender: Antivirus"
5. Let's add the permissions: Post Notifications, External Storage (Read), External Storage (Write) and Location Access (fine). This should be set to "Auto Grant"
6. Lets add the configuration keys: 
| Configuration Key                          | Value Type | Configuration Value         |
|--------------------------------------------|------------|-----------------------------|
| VPN                                        | integer    | 1                           |
| User UPN                                   | variable     | User Principal Name       |
| Low touch onboarding                       | integer    | 1                           |
| Disable sign out                           | integer    | 1                           |
| Enable Network Protection in Microsoft Defender | integer    | 1                           |
| Microsoft Defender                         | integer    | 1                           |
| Anti-Phishing                              | integer    | 1                           |

Push this policy to your users and save the policy.
**Info Notice:** When you save the profile the "User UPN" will say it's still on "string". This is a visual bug and it is still "variable" inside the JSON editor.
{: .notice--info}

# Push Defender
After you've pushed the Always-On VPN and App configuration policy you can push the Defender app. The users will get around 3 steps instead of the usual 6-7.