---
layout: post
title: Terraform Module To Deploy Cloud Shell With VNet Integration
---

I've written a reusable Terraform Module for easily deploying Cloud Shell VNet Integration.  

You can find the module in my GitHub repo [here](https://github.com/dsmithcloud/terraform-azurerm-cloudshell-vnet) or in the Terriform Registry [here](https://registry.terraform.io/modules/dsmithcloud/cloudshell-vnet/azurerm/latest).

This module is based on the source Microsoft-provided ARM template version [here](https://github.com/Azure/azure-quickstart-templates/blob/master/demos/cloud-shell-vnet/azuredeploy.json).

# Supported Regions
**Note**: Cloud Shell VNet integration is only supported in the following regions as of this writing:

'westus, southcentralus, eastus, northeurope, westeurope, centralindia, southeastasia, westcentralus, eastus2euap, centraluseuap'

# Usage

This module will deploy all of the requisite resources to support connecting cloud shell to your virtual network.  All resources are deployed into the same Resource Group as the existing virtual network.

To use this module you must have an existing vnet already deployed - or you can deploy one as part of your terraform repo - just reference that new vnet in the module parameters, and include a depends_on block.

The module deployes the following resource:

+ **Container Subnet** - adds an additional subnet to the existing vnet for the cloud shell ACI to connect to.  This subnet will have service endpoints enabled for "Microsoft.Storage" and will be delegated for "Microsoft.ContainerInstance/containerGroups".
+ **Relay Subnet** - adds an additonal subnet to the existing vnet for the relay namespace to connect to.
+ **Network Profile** - Creates a network profile and associated NIC and attaches it to the Container Subnet.
+ **Relay Namespace** - creates a relay namespace with a "Standard" SKU.
+ **Role Assignments** - Makes two RBAC role assignments:
    - _networkRoleDefinition_ - Assigns the Built-In networkRoleDefinition to the network profile that is created by the module.
    - _contributorRoleDefinition_ - Assigns the Built-In contributorRoleDefinition to the relay namespace that is created by the module.
+ **Private Endpoint** - Creates a Private Endpoint resource in the relay subnet that's created by this module.  It then is associated with the Relay Namespace that is also created by this module.
+ **Private DNS Zone** - Creates a private dns zone named 'privatelink.servicebus.windows.net' and links this zone to the existing virtual network.
    - _DNS A Record_ - Creates an A record in the newly created DNS zone that points to the IP address of the newly created private endpoint.
+ **Storage Account** - creates a storage account enabled for LRS and TLS1.2.  The storage account firewall is then enabled to only accept traffic from the Container Subnet created by this module.
+ **Tags** - This module will assign user-provided tags to all resources if included in the module block.  However, the storage account will always be assigned the following tags:
    - "ms-resource-usage" = "azure-cloud-shell"


## Example

{% gist a5799ba0265e125268f5475ea3fa32c8 module.tf %}

## Parameters

**source**: (Required): Location where the module is stored.  Use the github.com URL in the example above, or if you want to download a copy of the module separately, use the local path where the module is included with the rest of your terraform files.

**existing-vnet-name**: (Required) The name of the existing virtual network

**existing-vnet-resource-group**: (Required) The name of the resource containing the existing virtual network

**ACI-OID**: (Required) Azure Container Instance OID.  You can obtain this value by running the following command:

{% gist a5799ba0265e125268f5475ea3fa32c8 find-aci-oid.ps1 %}

**container-subnet-name**: (Optional) the name to be assigned to the cloudshell container subnet (default     = "cloudshellsubnet")

**container-subnet-prefix**: (Required) the list of address prefix(es) to be assigned to the cloudshell container subnet

**relay-subnet-name**: (Optional) the name to be assigned to the relay subnet (default     = "relaysubnet")

**relay-subnet-prefix**: (Required) the list of address prefix(es) to be assigned to the relay subnet

**relay-namespace-name**: (Optional) The name to be assigned to the relay namespace. Must be globally unique! (default     = "cshrelay")

**storageaccount-name**: (Required) the name of the storage account to create

**private-endpoint-name**: (Optional) the name to be assigned to the private endpoint (default     = "cloudshellRelayEndpoint")

**tags**: (Optional) the list of tags to be assigned (default     = {})

**depends_on**: (Optional) Include this if you are also creating the resource group and vnet in the same repo
