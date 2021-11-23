---
layout: post
title: Terraform Module To Deploy Cloud Shell With VNet Integration
---

I've written a reusable Terraform Module for easily deploying Cloud Shell VNet Integration.  

For the Microsoft-provided ARM template, see [this link](https://github.com/Azure/azure-quickstart-templates/blob/master/demos/cloud-shell-vnet/azuredeploy.json).

# Supported Regions
**Note**: Cloud Shell VNet integration is only supported in the following regions as of this writing:

'westus,southcentralus,eastus,northeurope,westeurope,centralindia,southeastasia,westcentralus,eastus2euap,centraluseuap'

# Usage

To use this module

## Example

```terraform
module "cloudshell-vnet" 
  source                       = "git::https://github.com/dsmithcloud/tf-cloudshell-vnet.git"
  existing-vnet-name           = "vnet-core-ussc-10.0.0.0_24"
  existing-vnet-resource-group = "rg-global-core-network"
  ACI-OID                      = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  container-subnet-prefix      = ["10.0.0.96/27"]
  relay-subnet-prefix          = ["10.0.0.128/26"]
  relay-namespace-name         = "cshrelay"
  storageaccount-name          = "storageacctname"
  tags                         = {"key"="value"}
  depends_on                   = [azurerm_resource_group.my-rg-name]
}
```

## Parameters

**source**: (Required): Location where the module is stored.  Use the github.com URL in the example above, or if you want to download a copy of the module separately, use the local path where the module is included with the rest of your terraform files.

**existing-vnet-name**: (Required) The name of the existing virtual network

**existing-vnet-resource-group**: (Required) The name of the resource containing the existing virtual network

**ACI-OID**: (Required) Azure Container Instance OID.  You can obtain this value by running the following command:

```powershell
Get-AzADServicePrincipal -DisplayNameBeginsWith 'Azure Container Instance'
```

**container-subnet-name**: (Optional) the name to be assigned to the cloudshell container subnet (default     = "cloudshellsubnet")

**container-subnet-prefix**: (Required) the list of address prefix(es) to be assigned to the cloudshell container subnet

**relay-subnet-name**: (Optional) the name to be assigned to the relay subnet (default     = "relaysubnet")

**relay-subnet-prefix**: (Required) the list of address prefix(es) to be assigned to the relay subnet

**relay-namespace-name**: (Optional) The name to be assigned to the relay namespace. Must be globally unique! (default     = "cshrelay")

**storageaccount-name**: (Required) the name of the storage account to create

**private-endpoint-name**: (Optional) the name to be assigned to the private endpoint (default     = "cloudshellRelayEndpoint")

**tags**: (Optional) the list of tags to be assigned (default     = {})

**depends_on**: (Optional) Include this if you are also creating the resource group and vnet in the same repo