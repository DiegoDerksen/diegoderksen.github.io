---
title: "Post: Compliancy e-mail HTML formatting"
last_modified_at: 2024-06-21T00:10:00
categories:
  - Blog
tags:
  - Intune
---

Sometimes, Microsoft comes with updates that aren't getting the attention it deserves. This is one of them. They have recently improved the HTML formatting for compliance messages within Intune. This update brings a cleaner, more structured look to these messages by introducing features such as paragraphs and variable content, including user names, device names, and OS versions.

In this blog post, we will explore how to set up and take advantage of these new HTML formatting features in Intune to ensure your compliance messages are not only more readable but also more informative. Let's dive into the details!


# Steps

## Create a message template

1. Go to intune.microsoft.com
2. Go to Devices > Compliance
3. Click in the top of the menu on "Notifications" and create a new notifcation
4. Choose a name and whether you want a header and footer
5. In the next step you can create a new message template with HTML formatting:  

The following tag are included in the editor:

```HTML
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

Besides these tags, there are also a few dynamic contents that you can use:  

![Table of variable](/assets/images/Compliancy-e-mail-HTML-formatting/Table-VarCont-Tokens.png)

To simplify the process, I used a What You See Is What You Get (WYSIWYG) HTML editor, such as [WYSIWYG HTML Editor](https://wysiwyghtml.com/). 

This tool is incredibly helpful, especially if you don't have any prior experience with HTML. After all, who doesn’t appreciate making things easier? If you want to use the variable content just use the token and paste it inside the HTML formatted text. You can even add links to the mail.

I have the following example, with a link:

```HTML
<p>Dear UserNameToken,</p>
<p>Our system indicates that the following device: DeviceNameToken currently does not comply with our company policy. If you do not follow up on this email, it will affect the proper functioning of the device.</p>
<p>The alert in our system can have various causes: missing software updates, security settings that do not meet the standard, or the device not being turned on for an extended period.</p>
<p>If you cannot resolve the issue yourself, please contact the IT Service Desk or register a ticket in the <a href="https://LinktoTicketSystem.com">ticket system</a>.</p>
<p><br />Kind regards,</p>
```

If you're finished with the template, you can save it. After this you can then add this template to a compliancy policy in the “Actions for noncompliance” section.  

Here is an example of the template:
![Template example](/assets/images/Compliancy-e-mail-HTML-formatting/HTML_Formatting_Example_email.png)

**Info Notice:** There is no {{Device Name}} from the dynamic content since it's a preview mail.
{: .notice--info}

# Conclusion

The recent improvement to HTML formatting for compliance messages in Intune is working amazing. This update significantly enhances the readability and informativeness of compliance messages by introducing structured HTML features and dynamic content variables.

By following the steps outlined in this blog post, you can easily set up and leverage these new HTML formatting features in Intune. From creating message templates to using a WYSIWYG HTML editor for convenience, we've covered all you need to know to make your compliance messages more effective and user-friendly.

Incorporating these enhancements will ensure your messages are clear, personalized, and actionable, ultimately contributing to better compliance and smoother operations within your organization.