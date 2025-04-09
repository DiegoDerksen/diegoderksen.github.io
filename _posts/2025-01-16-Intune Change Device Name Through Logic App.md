---
title: "Intune change device name through Logic App"
last_modified_at: 2025-01-16T21:30:00+02:00
categories:
  - Blog
tags:
  - Power Automate
  - Logic App
---

In our organization, we needed to change device names in Intune using a Logic App or Power Automate flow. We wanted to use the enrollmentProfileName to identify different device groups, but we discovered that querying it through the Graph API's list function returned Null values. As a workaround, we modified the DeviceName to match the enrollmentProfileName. This solution allows our application administrators to easily identify devices and push configurations effectively, even though we're working with userless devices and apps. I'll demonstrate this process using Zebra devices as an example.

# **Prerequisites:**

Power Automate Premium (for HTTP connector) or Azure Logic App

App registration in Azure with the following permission:

Graph API > DeviceManagementManagedDevices.Read.All (Application) Graph API > DeviceManagementManagedDevices.PrivilegedOperations.All

# **Make a App Registration in Azure**

1. Go to [portal.azure.com](http://portal.azure.com)
2. Navigate to Azure Active Directory
3. Select App Registration
4. Click New Registration
5. Enter a name for your app and select Register
6. Once created, go to API Permissions
7. Select Add a Permission
8. Choose Microsoft Graph, then Application Permissions
9. Search for and add these permissions: **DeviceManagementManagedDevices.Read.All** and **DeviceManagementManagedDevices.PrivilegedOperations.All**. After adding them, grant admin consent

# **Changing device name with a HTTP POST request**

1. Start with a recurrence trigger set to your preferred interval
2. Create 3 Initialize Variables steps for:
    1. ClientID from App Registration
    2. TenantID from Entra
    3. Audience with value: https://graph.microsoft.com
3. Get the certificate from Key Vault using the "Get secret" step
4. Retrieve all Zebra devices from Intune with an HTTP GET request. Use this URI: <code> https://graph.microsoft.com/BETA/deviceManagement/managedDevices?$select=id,deviceName&amp;$filter=manufacturer%20eq%20'Zebra%20Technologies'</code>. Set Authentication Type to Active Directory OAuth and include the 3 variables you created as dynamic content. Set Credential Type to Certificate and use the Value from the "Get secret" step as dynamic content, with password expression set to "Null"
    
    ![](https://diegoderksen.github.io/assets/images/LogicApps/HTTP%20GET%20all%20zebra%20devices.png)
    
5. Create a new step as “Parse JSON”. Content should be the body from step 4 and use the following sample payload:
    
    ```yaml
      {
     "type": "object",
     "properties": {
         "statusCode": {
             "type": "integer"
         },
         "headers": {
             "type": "object",
             "properties": {
                 "Transfer-Encoding": {
                     "type": "string"
                 },
                 "Vary": {
                     "type": "string"
                 },
                 "Strict-Transport-Security": {
                     "type": "string"
                 },
                 "request-id": {
                     "type": "string"
                 },
                 "client-request-id": {
                     "type": "string"
                 },
                 "x-ms-ags-diagnostic": {
                     "type": "string"
                 },
                 "OData-Version": {
                     "type": "string"
                 },
                 "Date": {
                     "type": "string"
                 },
                 "Content-Type": {
                     "type": "string"
                 },
                 "Content-Length": {
                     "type": "string"
                 }
             }
         },
         "body": {
             "type": "object",
             "properties": {
                 "@@odata.context": {
                     "type": "string"
                 },
                 "id": {
                     "type": "string"
                 },
                 "enrollmentProfileName": {
                     "type": "string"
                 },
                 "deviceName": {
                     "type": "string"
                 }
             }
         }
     }
      }
    
    ```
    
6. Create a For Each with Output: `@{outputs('Parse_JSON_')?['body']?['value']}`
7. Create a HTTP GET inside the for each loop to get the enrollment profile: `https://graph.microsoft.com/beta/deviceManagement/managedDevices/@{item()?['id']}?$select=id,enrollmentprofilename,deviceName`
    
    ![](https://diegoderksen.github.io/assets/images/LogicApps/HTTP%20GET%20EnrollmentProfileName.png)
    
8. Create a new step as “Parse JSON”. Content should be the body from step 7 and use the following sample payload:
    
    ```yaml
     {
     "type": "object",
     "properties": {
         "statusCode": {
             "type": "integer"
         },
         "headers": {
             "type": "object",
             "properties": {
                 "Transfer-Encoding": {
                     "type": "string"
                 },
                 "Vary": {
                     "type": "string"
                 },
                 "Strict-Transport-Security": {
                     "type": "string"
                 },
                 "request-id": {
                     "type": "string"
                 },
                 "client-request-id": {
                     "type": "string"
                 },
                 "x-ms-ags-diagnostic": {
                     "type": "string"
                 },
                 "OData-Version": {
                     "type": "string"
                 },
                 "Date": {
                     "type": "string"
                 },
                 "Content-Type": {
                     "type": "string"
                 },
                 "Content-Length": {
                     "type": "string"
                 }
             }
         },
         "body": {
             "type": "object",
             "properties": {
                 "@@odata.context": {
                     "type": "string"
                 },
                 "id": {
                     "type": "string"
                 },
                 "enrollmentProfileName": {
                     "type": "string"
                 },
                 "deviceName": {
                     "type": "string"
                 }
             }
         }
     }
     }
    
    ```
    
9. Add a condition step to check both the enrollment profile name and if the device needs renaming. In the condition expression, use these conditions with an AND operator: `@{body('Parse_zebra_enrollment_profile')?['enrollmentProfileName']}` equals YourEnrollmentProfile AND `@{item()?['deviceName']}` does not start with TEST-ZEBRA **(modify this to match your naming convention)**
10. Within the TRUE condition, add an HTTP POST request with this URI: `https://graph.microsoft.com/beta/deviceManagement/managedDevices/@{item()?['id']}/setDeviceName`. Set the Headers to "Content-type: application/json" and use this body:  `{ "deviceName": "TEST-ZEBRA-@{rand(100,5000)}" }`

![](https://diegoderksen.github.io/assets/images/LogicApps/HTTP%20POST%20RENAME%20DEVICE.png)

There you have it! You can now use Logic Apps, Power Automate, or even Postman to change device names in Intune.
