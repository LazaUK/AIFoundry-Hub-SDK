# Azure AI Studio: Creating Hub resource with Azure CLI
This repo explains how to use the **_Command Line Interface (CLI)_** to create an Azure AI Studio _hub_ resource. It shows how to deploy a Hub with auto-generated dependent resources like Azure Storage and Azure Key Vault, as well as how to create a new Hub and integrate it with already existing Storage and Key Vault.

> [!WARNING]
> You can create an Azure AI Studio hub with an existing Azure Storage account only if:
> - The Storage account uses the _Standard_ performance tier, not _Premium_,
> - Both _Blob_ and _File_ service capabilities are enabled,
> - If storage is of _Azure Data Lake Storage_ type, then its _Hierarchical namespace_ feature is disabled.

## Table of contents:
- [Pre-requisites](https://github.com/LazaUK/AIStudio-Hub-CLI#pre-requisites)
- [Scenario 1: Hub creation with bare minimum configuration](https://github.com/LazaUK/AIStudio-Hub-CLI#scenario-1-hub-creation-with-bare-minimum-configuration)
- [Scenario 2: Hub creation with existing dependent resources - CLI flags](https://github.com/LazaUK/AIStudio-Hub-CLI#scenario-2-hub-creation-with-existing-dependent-resources---cli-flags)
- [Scenario 3: Hub creation with existing dependent resources - YAML config](https://github.com/LazaUK/AIStudio-Hub-CLI#scenario-3-hub-creation-with-existing-dependent-resources---yaml-config)

## Pre-requisites
1. To follow these steps, you'll need:
    - An Azure subscription with an active Entra ID account,
    - An Azure Storage account (for Scenario 2 and 3),
    - An Azure Key Vault (for Scenario 2 and 3).
2. Use the ```az login``` command to authenticate with your Azure subscription using your Entra ID credentials.

## Scenario 1: Hub creation with bare minimum configuration
1. You can create an Azure AI Studio hub with the following Az CLI command by providing values for only two mandatory parameters: **Resource Group** and **Hub** name:
``` PowerShell
az ml workspace create --kind hub --resource-group <RESOURCE_GROUP_NAME> --name <AI_HUB_NAME>
```
2. As no other details are provided, Azure will automatically provision dependent **Azure Storage** and **Azure Key Vault** resources.

## Scenario 2: Hub creation with existing dependent resources - CLI flags
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

## Scenario 3: Hub creation with existing dependent resources - YAML config
1. Using long CLI commands with long Resource IDs as shown in Scenario 2 can be error-prone. You can define required values in YAML format instead, as per the attached ```Config_File.yaml``` file.
2. You can then create a new Hub resource with existing Storage and Key Vault resources using the following Azure CLI command:
``` PowerShell
az ml workspace create -g <RESOURCE_GROUP_NAME> --file <YAML_CONFIGURATION_FILE>
```
