---
title: "Tip: Windows 11 Local Account"
last_modified_at: 2024-06-03T22:00:00
categories:
  - Blog
tags:
  - Intune
---

Testing something in a virtual machine on my local Windows 11 setup proved to be tricky, especially when I encountered the requirement to log into a Microsoft account. This was frustrating because I needed to use a local account.  
Even disconnecting the network adapter didn't bypass this requirement. In this post, I'll show you the straightforward method I used to set up a local account without needing to log into a Microsoft account.

# Steps to Bypass Microsoft Account Requirement:

1. During the Out-of-box experience (OOBE), press Shift + F10 to open the Command Prompt.
2. Type the following command and press Enter: `OOBE\BYPASSNRO`. The PC will reboot.
3. Once the PC reboots and you're back in the OOBE, press Shift + F10 again to open the Command Prompt.
4. Type `ipconfig /release` and press Enter. This will disconnect your PC from the network.
5. Proceed with the installation and select "I don't have internet" followed by "Continue with limited setup".
6.  Complete the rest of the installation process.
7. Once you're on the desktop, open the Command Prompt again and type `ipconfig /renew` to reconnect to the network.

Congratulations! You now have a local account set up on your Windows 11 virtual machine.