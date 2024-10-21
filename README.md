# Azure AI Studio: Creating Hub resource with Azure Python SDK
This repo explains how to use the **_Azure Python Software Development Kit (SDK)_** to create an Azure AI Studio _hub_ resource. It demonstrates two scenarios: creating a Hub with bare minimum configuration (automatically provisioned storage and key vault), and creating a Hub with existing Azure Storage and Azure Key Vault resources.

> [!WARNING]
> You can create an Azure AI Studio hub with an existing Azure Storage account only if:
> - The Storage account uses the _Standard_ performance tier, not _Premium_,
> - Both _Blob_ and _File_ service capabilities are enabled,
> - If storage is of _Azure Data Lake Storage_ type, then its _Hierarchical namespace_ feature is disabled.

## Table of contents:
- [Pre-requisites](https://github.com/LazaUK/AIStudio-Hub-SDK#pre-requisites)
- [Scenario 1: Hub creation with bare minimum configuration](https://github.com/LazaUK/AIStudio-Hub-SDK#scenario-1-hub-creation-with-bare-minimum-configuration)
- [Scenario 2: Hub creation with existing dependent resources](https://github.com/LazaUK/AIStudio-Hub-SDK#scenario-2-hub-creation-with-existing-dependent-resources)

## Pre-requisites
1. To follow these steps, you'll need:
    - An Azure subscription with an active Entra ID account,
    - An Azure Storage account (for Scenario 2),
    - An Azure Key Vault (for Scenario 2),
    - Python 3.11 or later.
2. You can install the required ```Azure ML```, ```Azure Identity```, ```Azure Storage``` and ```Azure Key Vault``` Python SDK packages using pip command:
``` PowerShell
pip install -r requirements.txt
```

## Scenario 1: Hub creation with bare minimum configuration
1. To create a Hub resource in Azure, you need to initialise **MLClient** first. The ```DefaultAzureCredential``` class allows your code to check available identities in your environment and retrieve the relevant access token automatically:
``` Python
client = MLClient(
    credential = DefaultAzureCredential(),
    subscription_id = subscription_id,
    resource_group = resource_group
)
```
2. You can then instantiate the Hub object with desired property values, such as _name_, _resource_group_ and Azure region _location_:
``` Python
# Instantiate Hub object
ai_hub_config = Hub(
    name = ai_hub_name,
    display_name = ai_hub_display_name,
    description = ai_hub_description,
    location = ai_hub_location,
    resource_group = ai_hub_resource_group
)
```
3. You can create a new Hub resource by using the _begin_create()_ function of MLClient's _workspaces_ class with the previously created Python _Hub_ object:
``` Python
ai_hub = client.workspaces.begin_create(ai_hub_config).result()
```
4. If successful, you should see the function's output similar to this:
``` JSON
The deployment request Demo_AI_Hub_1-xxxxxx was accepted. ARM deployment URI for reference: <ARM_ID>
Creating Key Vault: (<Storage_Account_Name>  ) ..  Done (22s)
Creating Storage Account: (<Key Vault_Name>  )  Done (23s)
Creating AzureML Workspace: (Demo_AI_Hub_1  ) .....  Done (36s)
Total time : 1m 2s
```
5. As shown in the above step, the **Azure ML SDK** automatically provisioned dependent **Azure Storage** and **Azure Key Vault** resources. To get detailed information about the newly created Hub resoutrce, you can print its properties as shown below:
``` Python
ai_hub_json = json.dumps(ai_hub._to_dict(), indent=4)
print(ai_hub_json)
```
6. This should generate JSON output similar to this:
``` JSON
{
    "name": "Demo_AI_Hub_1",
    "location": "swedencentral",
    "id": <Hub_ARM_ID>,
    "resource_group": "AAA_AIHUB",
    "description": "Demo AI Hub with bare minimum configuration",
    "display_name": "Demo AI Hub 1",
    "hbi_workspace": false,
    "storage_account": <Storage_ARM_ID>,
    "key_vault": <KeyVault_ARM_ID>,
    "tags": {
        "createdByToolkit": "sdk-v2-1.21.1"
    },
    "public_network_access": "Enabled",
    "identity": {
        "type": "system_assigned",
        "principal_id": "XXX",
        "tenant_id": "YYY"
    },
    "managed_network": {
        "isolation_mode": "disabled",
        "outbound_rules": []
    },
    "enable_data_isolation": true,
    "default_resource_group": <Resource_Group>,
    "associated_workspaces": []
}
```

## Scenario 2: Hub creation with existing dependent resources
1. If you already have an existing **Azure Storage** account, you can retrieve its Resource ID either from the Azure portal, or by using the **Azure Management SDK for Storage**:
``` Python
storage_client = StorageManagementClient(
    credential = DefaultAzureCredential(),
    subscription_id = subscription_id
)

storage_account_id = storage_client.storage_accounts.get_properties(
    resource_group,
    ai_hub_storage_account
).id
```
> [!Note]
> The returned Storage account's **Resource ID** should look like this: _/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE_ACCOUNT>_
2. If you already have an existing **Azure Kay Vault**, you can retrieve its Resource ID either from the Azure portal, or by using the **Azure Management SDK for Key Vault**:
``` Python
key_vault_client = KeyVaultManagementClient(
    credential = DefaultAzureCredential(),
    subscription_id = subscription_id
)

key_vault_id = key_vault_client.vaults.get(
    resource_group,
    ai_hub_key_vault
).id
```
> [!Note]
> The returned Key Vault's **Resource ID** should look like this: _/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEY_VAULT>_
3. You can use the Resource IDs from the above steps to set the values of ```storage_account``` and ```key_vault``` properties of a _Hub_ resource:
``` Python
ai_hub_config = Hub(
    name = ai_hub_name,
    display_name = ai_hub_display_name,
    description = ai_hub_description,
    location = ai_hub_location,
    resource_group = ai_hub_resource_group,
    storage_account = storage_account_id, # Existing storage account's Resource ID
    key_vault = key_vault_id # Existing key vault's Resource ID
)
```
4. The next step would be to create a new Hub resource with the **Azure ML SDK**:
``` Python
ai_hub = client.workspaces.begin_create(ai_hub_config).result()
```
5. As our Hub's configuration references existing Storage and Key Vault resources, Azure will create only the Hub resource this time:
``` JSON
The deployment request Demo_AI_Hub_2-xxxxxx was accepted. ARM deployment URI for reference: <ARM_ID>
Creating AzureML Workspace: (Demo_AI_Hub_2  ) ......  Done (36s)
Total time : 37s
```
6. You can retrieve the Hub's properties with the following code:
``` Python
ai_hub_json = json.dumps(ai_hub._to_dict(), indent=4)
print(ai_hub_json)
```
7. The generated output will confirm that the newly created Hub is configured to use existing Storage and Key Vault resources:
``` JSON
{
    "name": "Demo_AI_Hub_2",
    "location": "swedencentral",
    "id": <Hub_ARM_ID>,
    "resource_group": "AAA_AIHUB",
    "description": "Demo AI Hub with existing dependent resources",
    "display_name": "Demo AI Hub 2",
    "hbi_workspace": false,
    "storage_account": <Storage_ARM_ID>,
    "key_vault": <KeyVault_ARM_ID>,
    "tags": {
        "createdByToolkit": "sdk-v2-1.21.1"
    },
    "public_network_access": "Enabled",
    "identity": {
        "type": "system_assigned",
        "principal_id": "XXX",
        "tenant_id": "YYY"
    },
    "managed_network": {
        "isolation_mode": "disabled",
        "outbound_rules": []
    },
    "enable_data_isolation": true,
    "default_resource_group": <Resource_Group>,
    "associated_workspaces": []
}
```
