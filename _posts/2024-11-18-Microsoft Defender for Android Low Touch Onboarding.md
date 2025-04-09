---
title: "Microsoft Defender for Android Low Touch Onboarding"
last_modified_at: 2024-11-18T20:00:00+02:00
categories:
  - Blog
tags:
  - Microsoft Defender
---

Microsoft has introduced **Low Touch Onboarding** (currently in preview) for Intune, a feature designed to streamline device setup and minimize IT intervention. By automating much of the enrollment process, it saves time, reduces effort, and significantly improves the end-user experience.

In this guide, I'll walk you through how to set up Low Touch Onboarding in Intune.

---

## Step 1: Add Microsoft Defender from Managed Google Play

The first step is to download **Microsoft Defender** from the Managed Google Play store.

1. Navigate to **Apps** in the Intune portal.
2. Go to **Android**.
3. Click **Add** and select **Managed Google Play app**.
4. Search for **Microsoft Defender: Antivirus**.
5. Select the app and click **Select**.

Don't push Defender yet to your users. We should do this as the last step.

---

## Step 2: Configure Always-On VPN for Microsoft Defender

To enable the Web Protection feature in Defender for Android, we need to set up an Always-On VPN. This isn’t a standard VPN—it’s a self-looping, local VPN that doesn’t route traffic outside the device.

1. Open the **Intune** portal.
2. Go to **Android configuration policies**.
3. Create a new policy and navigate to **Connectivity**.
4. Enable **Always-on VPN (work profile level)**.
5. Set **VPN Client** to **Custom**.
6. Enter the **Package ID** as `com.microsoft.scmx`.
7. Deploy this configuration to your users and save the policy.

This step eliminates the need for users to manually install the VPN, making onboarding even smoother.

---

## Step 3: Create App Configuration Policies

Next, configure an **App Configuration Policy** to enable features and activate Low Touch Onboarding.

1. Navigate to **Apps** in Intune.
2. Go to **App Configuration policies** and click **Create** > **Managed devices**.
3. Fill in the policy name and select:
   - **Platform**: Android Enterprise
   - **Profile type**: Fully Managed, Dedicated, and Corporate-Owned Work Profile Only
   - **Targeted app**: Microsoft Defender: Antivirus

4. Add the following permissions and set them to **Auto Grant**:
   - Post Notifications
   - External Storage (Read)
   - External Storage (Write)
   - Location Access (Fine)

5. Configure the following keys:

   | Configuration Key                          | Value Type | Configuration Value         |
   |--------------------------------------------|------------|-----------------------------|
   | VPN                                        | Integer    | 1                           |
   | User UPN                                   | Variable   | User Principal Name         |
   | Low touch onboarding                       | Integer    | 1                           |
   | Disable sign out                           | Integer    | 1                           |
   | Enable Network Protection in Microsoft Defender | Integer    | 1                           |
   | Microsoft Defender                         | Integer    | 1                           |
   | Anti-Phishing                              | Integer    | 1                           |

6. Deploy the policy to your users and save it.

**Info Notice:** When saving the policy, the **User UPN** field might still appear as "string" in the interface. This is a visual bug—the configuration is correctly saved as "variable" in the JSON editor.
{: .notice--info}

---

## Step 4: Push the Defender App

Once the Always-On VPN and App Configuration Policy are deployed, push the Microsoft Defender app. With Low Touch Onboarding, users only need to complete 3 steps instead of the usual 6-7, offering a significantly better experience.

---

With these steps, you’ve successfully implemented Low Touch Onboarding in Intune, streamlining the setup process and enhancing the user experience.
