---
title: "Post: Microsoft Defender for Endpoints reports to Excel"
last_modified_at: 2024-04-22T21:30:00+02:00
categories:
  - Blog
tags:
  - Power Automate
  - Microsoft Defender for Endpoint
  - Reporting
---

In this tutorial, I'll walk you through the process of setting up automated reporting from Microsoft Defender for Endpoint alerts directly to an Excel sheet using Power Automate and/or Logic App. By implementing this solution, you'll gain valuable insights into activities within your environment, simplifying monitoring and analysis tasks. Let's dive into the steps to streamline your reporting workflow and empower your security efforts

# Prerequisites:
Power Automate Premium (for HTTP connector)

App registration in Azure with the following permission:  
Graph API > SecurityAlert.Read.All (Application)

# Make a App Registration in Azure
1. Go to Azure.com
2. Click on Azure Active Directory
3. Click on App Registration
4. Click on New Registration
5. Create a new name and click on Register
6. After the app has been created go to API Permissions
7. Click on Add a Permission
8. Select Microsoft Graph and select Application Permissions
9. Look up the following permission SecurityAlert.Read.All and add this to the application followed up by granting admin consent

## Create a client secret
1. Go to your newly created app
2. Certificates & Secrets
3. Click on New Client Secret
4. Set a description and expiry to your liking
5. Copy the value of the secret, you **won't** be able to see this after you close this window

# Make the flow:
1. I've made a recurrent flow so it runs every Monday at 10 AM.
1. We need to initialize 4 variables so it can use the app registration in Azure
   
    | Name Variable | Name  | Type  | Value  |
    | ------------ | ------------ | ------------ | ------------ |
    | Initialize TenantID | TenantID | String | {Paste your Tenant ID} |
    | Initialize ClientID | ClientID | String | {Paste your Client ID} |
    | Initialize Audience | Audience | String | https://graph.microsoft.com |
    | Initialize Secret |  Secret | String | {Paste your Secret} |

    Your flow should look something like this:
    ![Initialize](/assets/images/PA-MDE-alerts-to-Excel/Initialize%20flow.png)

1. Make a new step with a HTTP connector with the following settings:  
    Method: Get  
    URI: `https://graph.microsoft.com/v1.0/security/alerts_v2?$filter=servicesource+eq+'microsoftDefenderForEndpoint'+and+createdDateTime+ge+@{formatDateTime(subtractFromTime(utcNow(), 7, 'Day'), 'yyyy-MM-ddTHH:mm:ssZ')}`  
    this URI will get all alerts from Microsoft Defender for Endpoint in the last 7 days

    Authentication: Active Directory OAuth

    Put in the variables from step 2 inside Tenant, Audience, Client ID and Secret

    The step should look like this:
    
    ![HTTP Connector](/assets/images/PA-MDE-alerts-to-Excel/HTTP%20Connector.png)


1. Make a new step with a Parse JSON variable
    Content should be:
    <details>
    <summary>Body Schema</summary>

    `{ "type": "object", "properties": { "statusCode": { "type": "integer" }, "headers": { "type": "object", "properties": { "Transfer-Encoding": { "type": "string" }, "Vary": { "type": "string" }, "Strict-Transport-Security": { "type": "string" }, "request-id": { "type": "string" }, "client-request-id": { "type": "string" }, "x-ms-ags-diagnostic": { "type": "string" }, "OData-Version": { "type": "string" }, "Date": { "type": "string" }, "Content-Type": { "type": "string" }, "Content-Length": { "type": "string" } } }, "body": { "type": "object", "properties": { "@@odata.context": { "type": "string" }, "value": { "type": "array", "items": { "type": "object", "properties": { "id": { "type": "string" }, "providerAlertId": { "type": "string" }, "incidentId": { "type": "string" }, "status": { "type": "string" }, "severity": { "type": "string" }, "classification": {}, "determination": {}, "serviceSource": { "type": "string" }, "detectionSource": { "type": "string" }, "productName": { "type": "string" }, "detectorId": { "type": "string" }, "tenantId": { "type": "string" }, "title": { "type": "string" }, "description": { "type": "string" }, "recommendedActions": { "type": "string" }, "category": { "type": "string" }, "assignedTo": { "type": "string" }, "alertWebUrl": { "type": "string" }, "incidentWebUrl": { "type": "string" }, "actorDisplayName": {}, "threatDisplayName": { "type": "string" }, "threatFamilyName": { "type": "string" }, "mitreTechniques": { "type": "array" }, "createdDateTime": { "type": "string" }, "lastUpdateDateTime": { "type": "string" }, "resolvedDateTime": { "type": "string" }, "firstActivityDateTime": { "type": "string" }, "lastActivityDateTime": { "type": "string" }, "systemTags": { "type": "array" }, "alertPolicyId": {}, "additionalData": {}, "comments": { "type": "array" }, "evidence": { "type": "array", "items": { "type": "object", "properties": { "@@odata.type": { "type": "string" }, "createdDateTime": { "type": "string" }, "verdict": { "type": "string" }, "remediationStatus": { "type": "string" }, "remediationStatusDetails": {}, "roles": { "type": "array" }, "detailedRoles": { "type": "array", "items": { "type": "string" } }, "tags": { "type": "array" }, "firstSeenDateTime": { "type": "string" }, "mdeDeviceId": { "type": "string" }, "azureAdDeviceId": { "type": "string" }, "deviceDnsName": { "type": "string" }, "osPlatform": { "type": "string" }, "osBuild": { "type": "integer" }, "version": { "type": "string" }, "healthStatus": { "type": "string" }, "riskScore": { "type": "string" }, "rbacGroupId": { "type": "integer" }, "rbacGroupName": {}, "onboardingStatus": { "type": "string" }, "defenderAvStatus": { "type": "string" }, "ipInterfaces": { "type": "array", "items": { "type": "string" } }, "vmMetadata": {}, "loggedOnUsers": { "type": "array" }, "detectionStatus": { "type": "string" }, "fileDetails": { "type": "object", "properties": { "sha1": { "type": "string" }, "sha256": { "type": "string" }, "fileName": { "type": "string" }, "filePath": { "type": "string" }, "fileSize": { "type": "integer" }, "filePublisher": {}, "signer": {}, "issuer": {} } } }, "required": [ "@@odata.type", "createdDateTime", "verdict", "remediationStatus", "remediationStatusDetails", "roles", "detailedRoles", "tags", "mdeDeviceId" ] } } }, "required": [ "id", "providerAlertId", "incidentId", "status", "severity", "classification", "determination", "serviceSource", "detectionSource", "productName", "detectorId", "tenantId", "title", "description", "recommendedActions", "category", "assignedTo", "alertWebUrl", "incidentWebUrl", "actorDisplayName", "threatDisplayName", "threatFamilyName", "mitreTechniques", "createdDateTime", "lastUpdateDateTime", "resolvedDateTime", "firstActivityDateTime", "lastActivityDateTime", "systemTags", "alertPolicyId", "additionalData",     "comments", "evidence" ] } } } } } }`

    </details>

1. Create a new step with Create File in Sharepoint

    Choose a site address

    Choose the folder where you want to create the file

    File name should be: `Report - Anti Virus Alerts @{formatDateTime(utcNow(), 'dd-MM-yyyy')}.xlsx`

    File Content: use a expression with: Null

1. Create a new step to create table
    Location should be the same location as step 6

    Document Library: Documents

    File:  `@outputs('Create_file_in_Sharepoint')?['body/Id']`

    Table Range: A1:F1

    Column names: `DeviceName,title,severity,category,status,createdtime`

1. Create a Apply to Each step
   Ouput should be: `@{body('Parse_JSON')?['value']}`  
   Create another Apply to Each step inside this, this ouput should be: `@{items('Apply_to_each')['evidence']}`

1. Create a Add a row into a table step inside Apply to Each 2
Location: Same as in step 6

Document Library: Documents

File: `@outputs('Create_file_in_Sharepoint')?['body/Id']`

Table: `@outputs('Create_table')?['body/name']`

Row:
`{
  "DeviceName": @{items('Apply_to_each_2')?['deviceDnsName']},
  "title": @{items('Apply_to_each')?['title']},
  "severity": @{items('Apply_to_each')?['severity']},
  "category": @{items('Apply_to_each')?['category']},
  "status": @{items('Apply_to_each')?['status']},
  "createdtime": @{items('Apply_to_each')?['createdDateTime']}
}
`

It should look like this: 
![Apply to Each](/assets/images/PA-MDE-alerts-to-Excel/Apply%20to%20Each.png)


# End result screenshot

**Info Notice:** There is a "clean excel file" step. However this isn't needed if expression is set at Null in step 5.
{: .notice--info}

![Result](/assets/images/PA-MDE-alerts-to-Excel/Result%20Flow.png)