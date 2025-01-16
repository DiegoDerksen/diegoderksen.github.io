---
title: "Post: Intune change device name through Logic App"
last_modified_at: 2025-01-16T21:30:00+02:00
categories:
  - Blog
tags:
  - Power Automate
  - Logic App
---

In this quick post, I'll guide you through the process of changing device names in Intune using a Logic App or Power Automate flow. Our goal was to utilize the enrollmentProfileName, but since it returns Null when queried as a list through the Graph API, we opted to modify the DeviceName to match the enrollmentProfileName. This approach ensures our application administrators can identify devices and push configurations effectively, even though the devices and apps are userless. For this demonstration, I'll be using Zebra devices as an example.



# Prerequisites:
Power Automate Premium (for HTTP connector)

App registration in Azure with the following permission:  
Graph API > DeviceManagementManagedDevices.Read.All (Application)
Graph API > DeviceManagementManagedDevices.PrivilegedOperations.All

# Make a App Registration in Azure
1. Go to portal.azure.com
2. Click on Azure Active Directory
3. Click on App Registration
4. Click on New Registration
5. Create a new name and click on Register
6. After the app has been created go to API Permissions
7. Click on Add a Permission
8. Select Microsoft Graph and select Application Permissions
9. Look up the following permission DeviceManagementManagedDevices.Read.All & DeviceManagementManagedDevices.PrivilegedOperations.All and add this to the application followed up by granting admin consent

# [Follow my Azure Keyvault tutorial on how to use a certificate and keyvault with logic apps.](https://diegoderksen.github.io/blog/Securely-use-Azure-Logic-Apps-and-Azure-Keyvault/)

# Changing device name with a HTTP POST request

1. Start with the recurrence to your liking
2. Create 3 Initialize Variables steps for
    1. ClientID App Registration
    2. TenantID from Entra
    3. Audience with value: https://graph.microsoft.com
3. Get the certificate from Keyvault with the step "Get secret"
4. Now we want to get all Zebra devices from Intune with a HTTP GET request. Use the following URI: https://graph.microsoft.com/BETA/deviceManagement/managedDevices?$select=id,deviceName&$filter=manufacturer%20eq%20'Zebra%20Technologies'
Also be sure to use Authentication Type as Active Directory OAuth and use dynamic content to put in the 3 variables you've created earlier. The use Credential Type as Certificate and use the Value from the "Get secret" setp as dynamic content with password as expression "Null"
 ![HTTP GET all zebra devices.png](/assets/images/LogicApps/HTTP%20GET%20all%20zebra%20devices.png)
5. Create a new step as "Parse JSON". Content should be the body from step 4 and use the following sample payload:  
  ```JSON
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
6. Create a For Each with Output: `@{outputs('Parse_JSON_')?['body']?['value']}`
7. Create a HTTP GET inside the for each loop to get the enrollment profile: `https://graph.microsoft.com/beta/deviceManagement/managedDevices/@{item()?['id']}?$select=id,enrollmentprofilename,deviceName`
 ![HTTP GET EnrollmentProfileName.png](/assets/images/LogicApps/HTTP%20GET%20EnrollmentProfileName.png)
8. Create a new step as "Parse JSON". Content should be the body from step 7 and use the following sample payload:
    ```JSON
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
9. Use a condition step. This will look at the enrollmentprofile name and if the device has not yet been renamed. Inside the condition expression use the following conditions with a AND statement: `@{body('Parse_zebra_enrollment_profile')?['enrollmentProfileName']}` is equal to YourEnrollmentProfile AND `@{item()?['deviceName']}` Does not start with TEST-ZEBRA (Change to your own naming scheme)
10. Use a HTTP POST request inside the TRUE condition with the following URI: `https://graph.microsoft.com/beta/deviceManagement/managedDevices/@{item()?['id']}/setDeviceName` With Headers: Content-type application/json and body:  
` {
  "deviceName": "TEST-ZEBRA-@{rand(100,5000)}"
}`
 ![HTTP POST RENAME DEVICE.png](/assets/images/LogicApps/HTTP%20POST%20RENAME%20DEVICE.png)