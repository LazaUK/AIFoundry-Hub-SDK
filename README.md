# Azure AI Studio: Creating Hub resource with Azure Python SDK
This repo explains how to use the **_Azure Python Software Development Kit (SDK)_** to create an Azure AI Studio _hub_ resource. It demonstrates two scenarios:
1. Creating a Hub with bare minimum configuration (automatically provisioned storage and key vault).
2. Creating a Hub with existing Azure Storage and Azure Key Vault resources.

> [!WARNING]
> You can create an Azure AI Studio hub with an existing Azure Storage account only if:
> - The Storage account uses the _Standard_ performance tier, not _Premium_,
> - Both _Blob_ and _File_ service capabilities are enabled,
> - If storage is of _Azure Data Lake Storage_ type, then its _Hierarchical namespace_ feature is disabled.

## Table of contents:
- [Pre-requisites]()
- [Scenario 1: Hub creation with bare minimum configuration]()
- [Scenario 2: Hub creation with existing dependent resources]()

## Pre-requisites
1. To follow these steps, you'll need:
    - An Azure subscription with an active Entra ID account,
    - An Azure Storage account (for Scenario 2),
    - An Azure Key Vault (for Scenario 2),
    - Python 3.11 or later.
2. You can install the required ```Azure ML SDK```, ```Azure Identity```, ```Azure Storage``` and ```Azure Key Vault``` Python packages using pip:
``` PowerShell
pip install -r requirements.txt
```

## Scenario 1: Hub creation with bare minimum configuration
1. You can create an Azure AI Studio hub with the following Az CLI command by providing values for only two mandatory parameters: **Resource Group** and **Hub** name:
``` PowerShell
az ml workspace create --kind hub --resource-group <RESOURCE_GROUP_NAME> --name <AI_HUB_NAME>
```
2. As no other details are provided, Azure will automatically provision dependent **Azure Storage** and **Azure Key Vault** resources.

## Scenario 2: Hub creation with existing dependent resources
1. If you already have an existing **Azure Storage** account, you can retrieve its Resource ID either from the Azure portal, or by using the following Az CLI command:
``` PowerShell
az storage account show --name <STORAGE_NAME> --query "id"
```
> [!Note]
> The returned Storage account's **Resource ID** should look like this: _/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE_ACCOUNT>_
2. If you already have an existing **Azure Kay Vault**, you can retrieve its Resource ID either from the Azure portal, or by using the following Az CLI command:
``` PowerShell
az keyvault show --name <KEYVAULT_NAME> --query "id"
```
> [!Note]
> The returned Key Vault's **Resource ID** should look like this: _/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEY_VAULT>_
3. You can use the Resource IDs from the above steps to specify existing Storage and Key Vault as the default dependent resources for the new Hub resource in the following Azure CLI command:
``` PowerShell
az ml workspace create --kind hub --resource-group <RESOURCE_GROUP_NAME> --name <HUB_NAME> --display-name <HUB_DISPLAY_NAME> --description <HUB_DESCRIPTION> --location <HUB_AZURE_REGION> --storage-account <STORAGE_RESOURCE_ID> --key-vault <KEYVAULT_RESOURCE_ID>
```
