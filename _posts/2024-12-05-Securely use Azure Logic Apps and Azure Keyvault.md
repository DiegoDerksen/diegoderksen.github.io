---
title: "Securely use Azure Logic Apps and Azure Keyvault"
last_modified_at: 2024-12-05T11:00:00+02:00
categories:
  - Blog
tags:
  - Azure
  - Logic App
---

In this guide, I'll walk you through the process of setting up and managing Logic App resources using Role-Based Access Control (RBAC) and Managed Identities. We'll also cover the prerequisites.

## Prerequisites

- **Least privilege RBAC Roles in Resource Group**:
  - **Role**: Contributor
    - **What it does**: Create resources
  - **Role**: Key Vault Administrator
    - **What it does**: Create certificates
   - **Role**: RBAC Administrator
     - **What it does**: RBAC admin on resources

- **App Registration**
- **Azure Subscription**
- **Resource Groups**

## Steps

### Step 1: Create Key Vault

1. Navigate to the Azure portal.
2. Select "Resource groups" and choose the appropriate group.
3. Click "Create" and search for "Key Vault".
4. Fill in the required information:
   - **Key vault name**: xxx-KeyVault
   - **Region**: YourOwnRegion
   - **Pricing tier**: Standard
   - **Purge protection**: Enabled
   - Access configuration and Networking will be addressed in later steps.
5. Click "Review + Create" to create the Key Vault.

### Step 2: Generate and Download Certificate in Key Vault

1. Go to the created Key Vault and select "Certificates".
2. Click "Generate/Import" and choose "Generate".
3. Fill in the certificate details:
   - **Name**: App registration name
   - **Subject**: CN=example.com
4. Click "Create".
5. After generating, select the certificate and click "Download in CER format".

### Step 3: Upload Certificate to App Registration

1. Navigate to Azure Active Directory and select "App registration".
2. Choose the registered app and go to "Certificates & secrets".
3. Click "Upload certificate" and select the downloaded certificate.
4. Click "Add" to upload the certificate.

### Step 4: Create Logic App

1. Navigate to the Azure portal.
2. Select "Resource groups" and choose the appropriate group.
3. Click "Create" and search for "Logic App".
4. Choose "Consumption – Multi-tenant".
5. Fill in the following information:
   - **Logic app name**: Choose a clear name
   - **Region**: YourOwnRegion
   - **Enable log analytics**: No
6. Click "Review + Create".

### Step 5A: Copy Logic App Outgoing IP Addresses (One-time)

1. Go to the created Logic App.
2. Click "Properties".
3. Go to "Outgoing IP Addresses".
4. Copy the "Connector Outgoing IP addresses".

### Step 5B: Secure Key Vault – Firewall

1. Go to the Key Vault and select "Networking".
2. Click "Allow public access from specific virtual networks and IP addresses".
3. Add the IP addresses (one by one) to the "Firewall IP ranges".
   - If you want to manage the Key Vault via the Azure Portal, you must also whitelist your own public IP. To find your public IP: go to the command prompt and type in: curl ifconfig.me.

### Step 6A: Create User-Managed Identity

1. Navigate to the resource group and select "Create".
2. Search for "User Assigned Managed Identity" and select it.
3. Fill in the required information:
   - **Region**: YourOwnRegion
   - **Name**: MI-KEYVAULTNAME
4. Click "Review + Create".
5. Click "Create" to create the managed identity.

### Step 6B: User-Managed Identity Role Assignment

1. Go to the Managed Identity.
2. Go to "Azure Role Assignments".
3. Click "Add role assignment (preview)".
4. Choose the following:
   - **Scope**: Key Vault
   - **Subscription**: YourSub
   - **Resource**: The appropriate Key Vault for permissions
   - **Role**: Key Vault Secrets User
   - Note: You need RBAC administrator rights for this.

### Step 7: Add User-Managed Identity to Logic App

1. Go to the Logic App and select "Identity" under "Settings".
2. Enable "User-assigned" and click "Add".
3. Select the created user-managed identity and click "Add".
4. Click "Save" to save the changes.

## How to Use the Key Vault to Access Secrets via Managed Identity

1. Go to the created Logic App.
2. Go to "Logic App Designer".
3. Add the following steps:
   - **Step**: Variable
     - **Name**: ClientID
     - **Value**: ClientID App registration
   - **Step**: Variable
     - **Name**: TenantID
     - **Value**: YourTenantID
   - **Step**: Variable
     - **Name**: Audience
     - **Value**: https://graph.microsoft.com
4. Add the step "Get secret" from Azure Key Vault with the following information:
   - **Connection Name**: MI-KEYVAULTNAME-Keyvault
   - **Authentication Name**: Managed Identity
   - **Vault Name**: The exact Key Vault name
   - Select "Settings" of the "Get Secret" step and enable "Secure Outputs".
