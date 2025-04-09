---
title: "Intune Compliance Notifications Compliance HTML Formatting"
last_modified_at: 2025-04-09T21:00:00+02:00
categories:
  - Blog
tags:
  - Intune
---

Microsoft has made an important update to Intune that deserves more attention. They've made compliance messages look better and easier to read by adding new features. Now you can include paragraphs and personalized information like user names, device names, and operating system versions in these messages.

This guide will show you how to use these new features to create better compliance messages in Intune.

# **Steps**

## **Create a message template**

1. Go to [intune.microsoft.com](http://intune.microsoft.com)
2. Go to Devices > Compliance
3. Click "Notifications" at the top of the menu and create a new notification
4. Pick a name and decide if you want to add a header and footer
5. Now you can create your message using HTML formatting:

You can use these basic HTML tags in your message:

```html
<a>
<strong>
<b>
<u>
<ul>
<li>
<p>
<br>
<code>
<table>
<tbody>
<tr>
<td>
<thread>
<th>

```

You can also add these personalized details to your message:

| Variable Name     | Token to use               |
| ----------------- | -------------------------- |
| User Name         | &#123;&#123;UserName&#125;&#125;       |
| Device Name       | &#123;&#123;DeviceName&#125;&#125;     |
| Device ID         | &#123;&#123;DeviceId&#125;&#125;       |
| Device OS Version | &#123;&#123;OSAndVersion&#125;&#125;   |

To make writing easier, I recommend using a simple HTML editor called WYSIWYG HTML Editor ([available here](https://wysiwyghtml.com/)).

This tool makes creating messages much easier, especially if you don't know HTML. You can simply write your message and add the personalized details (like {{UserName}}) wherever you need them. You can even add clickable links to your message.

Here's a sample message you can use:

```html
<p>Dear {{UserName}},</p>
<p>Our system indicates that the following device: {{DeviceName}} currently does not comply with our company policy. If you do not follow up on this email, it will affect the proper functioning of the device.</p>
<p>The alert in our system can have various causes: missing software updates, security settings that do not meet the standard, or the device not being turned on for an extended period.</p>
<p>If you cannot resolve the issue yourself, please contact the IT Service Desk or register a ticket in the <a href="https://LinktoTicketSystem.com">ticket system</a>.</p>
<p><br />Kind regards,</p>

```

After you finish your template, save it. You can then use this template in any compliance policy by adding it to the "Actions for noncompliance" section.

Here's what the message looks like:

![](https://diegoderksen.github.io/assets/images/Compliancy-e-mail-HTML-formatting/HTML_Formatting_Example_email.png)

**Note**: The Device Name isn't shown here because this is just a preview.
{: .notice--info}

# **Conclusion**

The new HTML formatting in Intune is a great improvement. It helps you create clearer, better-looking compliance messages that include personalized information for each user.

Using this guide, you can easily create your own message templates in Intune. Whether you use the HTML editor or write the code yourself, you now have the tools to make your compliance messages clear and user-friendly.
