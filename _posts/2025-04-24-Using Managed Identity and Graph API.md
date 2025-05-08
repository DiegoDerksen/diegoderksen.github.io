---
title: "Using Managed Identity and Graph API"
last_modified_at: 2025-04-24T13:33:00+02:00
categories:
  - Blog
tags:
  - Azure
---


Using a managed identity instead of a service principal is a smart and secure idea. For example, you won't have to keep an eye out for secrets that are getting expired and it's a lot more secure. Even if your service principal uses a certificate instead of a client secret. This especially works well if you have business solutions in a Azure Logic App that needs to keep running 24/7 without having the scenario where the client secret expires and you only know it in the next morning if you don't monitor for it.

You also do not need to setup a seperate Azure Key Vault. Which saves administrative overhead. 

So, all in all just use a managed identity.

Before we dive into the technical setup, let's understand why you might choose managed identity over a service principal:

**Managed Identity vs Service Principal:**

- No credential management: Unlike service principals, managed identities don't require you to store or rotate secrets
- Automatic credential rotation: Azure Platform handles all credential management automatically, so your Logic Apps will run even if Microsoft rotates IP-addresses or when your client secrets are expired
- Simplified security: Reduced risk of credential exposure since there are no secrets to manage
- Cost-effective: No additional costs for managed identities, whereas maintaining service principals requires operational overhead
- Tight integration: Better integrated with Azure services and easier to implement

Now that we understand the benefits of managed identity, let's proceed with the implementation steps.

### **Step 1A: Initial Setup - Creating Azure Logic App**

1. Create a new Azure Logic App:
    - Go to the Azure Portal and click "Create a resource"
    - Search for "Logic App" and select it
    - Fill in the basic details:
        - Subscription: Select your subscription
        - Resource Group: Create new or select existing
        - Logic App name: Choose a unique name
        - Region: Select your preferred region
        - Plan type: Consumption (serverless)
    - Click "Review + Create" and then "Create"

### **Step 1B: Initial Setup - Using Managed Identity**

1. Enable managed identity on your Azure Logic App:
    - Navigate to the Logic App in Azure Portal
    - Go to "Identity" under Settings
    - Switch "Status" to "On" under System assigned
    - Click "Save"
    - Copy the Object ID of the managed identity
2. Grant API permissions to the managed identity:
    - Install and run the following PowerShell script (replacing the placeholder values):
    
    You can find all the permission names on the following [link](https://graphpermissions.merill.net/permission/)
   {: .notice--info}

     ```powershell
        Install-Module Microsoft.Graph -Scope CurrentUser
        
        Connect-MgGraph -Scopes Application.Read.All, AppRoleAssignment.ReadWrite.All, RoleManagement.ReadWrite.Directory
        
        $managedIdentityId = "<Managed Identity Object ID>"
        $roleName = "GraphAPI Placeholder Permission"
        
        $msgraph = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"
        $role = $Msgraph.AppRoles| Where-Object {$_.Value -eq $roleName} 
        
        New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $managedIdentityId -PrincipalId $managedIdentityId -ResourceId $msgraph.Id -AppRoleId $role.Id
         
        Disconnect-MgGraph
      ```
        
    - Verify the permissions are granted in Azure AD Enterprise Applications
    
   Info Notice: Switch the filtering from enterprise applications to managed identity
    {: .notice--info}

4. Configure your Logic App to use managed identity:
    - In your HTTP action, select "Managed Identity" as authentication type
    - Choose the system-assigned managed identity
    - Set audience to "https://graph.microsoft.com”
