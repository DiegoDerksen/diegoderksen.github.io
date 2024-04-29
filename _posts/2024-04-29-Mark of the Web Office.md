---
title: "Post: Mark of the Web Office"
last_modified_at: 2024-04-29T21:30:00
categories:
  - Blog
tags:
  - Microsoft
  - Office
  - Microsoft 365
---
I encountered a user who raised an incident because multiple departments were unable to open an Excel sheet with macros (.xlsm) from our Sharepoint site. The solution seemed simple: just right-click on the file, go to properties, and unblock it. Yet, it wasn't enough because they always used the same documents. This prompted me to delve deeper into what exactly occurs when someone downloads a file.
# What happens when someone downloads a file from the internet?
When a file is downloaded from the internet, it receives what's known as a "Mark of the Web" (MotW). This feature, inherent to NTFS, indicates the file's origin from the internet and warns of potential harm it could pose to the computer. Understanding the implications of this marker is crucial for maintaining the security, integrity and usability of your system.

# Why this could be a issue
Considering security, the "Mark of the Web" (MotW) serves as a valuable feature, [notwithstanding certain vulnerabilities that have been identified](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-36584). However, in an enterprise organization where departments frequently utilize macro documents shared on SharePoint, this feature can become a source of frustration. It highlights the need for a nuanced approach to security measures that balances protection with user convenience, particularly in workflow-intensive environments.

# When did Microsoft change this behaviour in Office?
Actually, several years ago, in Version 2208, which was released on October 11, 2022,

# What is the trusted sites for our OneDrive and Sharepoint?
As per [Microsoft documentation](https://learn.microsoft.com/en-us/deployoffice/security/internet-macros-blocked#files-on-onedrive-or-sharepoint), the correct URLs for SharePoint and OneDrive are: https://{your-domain-name}.sharepoint.com (for SharePoint) or https://{your-domain-name}-my.sharepoint.com (for OneDrive). However, I encountered an issue when attempting to use these URLs. They didn't work when saving it as a copy. Strangely, they did work when directly downloading it from SharePoint without opening the file or from Teams. This inconsistency raises questions about the underlying mechanics—why does this happen?

# Solving the issue
Let's explore the zone identifier assigned to the file using PowerShell. Navigate to the folder containing the file, then open PowerShell there and execute the following command (you don't need admin rights):

```powershell
get-content ".\filename.xlsm" -Stream zone.identifier
```
This command retrieves and displays the zone identifier associated with the specified file.

If you experiment with both options—downloading as a copy from within the file itself and downloading directly from SharePoint without opening the file—you'll notice two different URLs.

When saved as a "download" from SharePoint without opening the file, you'll receive: https://{your-domain-name}.sharepoint.com, as above. However, if saved as a copy, you'll get: https://euc-excel.officeapps.live.com. This disparity in URLs suggests varying mechanisms or pathways for file retrieval, potentially impacting the file's zone identifier and subsequent behavior.

# How could we fix this issue for internal documents?
Now that we know the correct URL's, let's delve into Intune. We need to create a new configuration policy utilizing the administrative template. Within the administrative templates, locate "Site to Zone assignments list." Here, you can designate a name (URL/IP) and assign it to a zone ranging from 1 to 4:

    1 Intranet
    2 Trusted sites
    3 Internet
    4 Restricted sites

Selecting zone 2, trusted sites, allows you to trust downloads from specific sites.

Add the following values:

| Value Name                                | Value |
| ----------------------------------------- | ----- |
| https://{your-domain-name}.sharepoint.com | 2     |
| https://euc-excel.officeapps.live.com     | 2     |

Ensure to assign the configuration policy the correct scope tag if you intend to use it, and assign it to the appropriate departments. This step ensures that the policy is applied only to the intended devices or users, enhancing organizational efficiency and security by targeting specific areas as needed.

# Conclusion
After exploring the intricacies of managing security measures in enterprise environments, it's evident that maintaining a balance between robust protection and user convenience is paramount. By understanding features like the "Mark of the Web" (MotW), navigating platform-specific URLs, and implementing precise policy configurations, organizations can effectively safeguard their assets while ensuring seamless workflows for users.

Remember to continuously evaluate and adapt security protocols based on emerging challenges and user feedback. Ultimately, a multifaceted approach to security management, combining technical expertise, user-centric design, and proactive policy implementation, is essential for safeguarding organizational assets in today's dynamic digital landscape.

Thank you for joining me on this exploration of security considerations in file management and user experience. 