---
layout: post
title: Prestage Azure Cloud Shell Resources and User Settings via API
---
## The Problem

One of my customers recently asked me if it's possible to pre-stage the required resources for Azure Cloud Shell ahead of time for a large number of users. They wanted to make Cloud Shell available to them, but they didn't want the users to have to understand how to create a storage account and file share, or how to connect a Cloud Shell session to a virtual network.

## The Solution

### Part 1: The Storage Account and File Share

Pre-staging the Storage Account and File share is the easy part. That can be done via any Infrastructure-as-Code (IaC) tools. Think Bicep, Terraform, ARM Templates, etc. There are plenty of examples of this on the web.  

Microsoft provides an ARM Template to do this in their GitHub quickstarts repo [here](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.storage/storage-file-share).

UPDATE: I wrote a [Terraform Module](https://registry.terraform.io/modules/dsmithcloud/cloudshell-vnet/azurerm/latest) to do this for you!

### Part 2: The User Settings

The User Settings for Cloud Shell get a little trickier however.

If a user has never opened a Cloud Shell session, they will be prompted to select Bash or PowerShell and a subscription to use.

![CloudShell-ChooseShell](https://static1.squarespace.com/static/615dc349c2707f51b61f219f/t/615e0515ec87f70bed005198/1633551637480/CloudShell-ChooseShell.png)

They can also choose advanced settings to select a specific pre-existing storage account and file share if they like as well.

![CloudShell-ChooseStorage](https://static1.squarespace.com/static/615dc349c2707f51b61f219f/t/615e057853563e57c432fc7a/1633551736618/CloudShell-ChooseStorage.png)

That doesn't solve my customer's request though. They don't want the user to have to make any of these choices.

Microsoft [documentation](https://docs.microsoft.com/en-us/azure/cloud-shell/troubleshooting#personal-data-in-cloud-shell) provides a way to query what these settings currently are via an API call. The documentation doesn't provide an answer for how you can SET those settings via an API, though.

You can use the Azure CLI to configure the User Settings using the `az rest` command.

First, create a JSON file with your desired settings using the samples below.

**Update: You can now get the files shown below from my [GitHub repo](https://github.com/dsmithcloud/update-cloudshell-via-api)!**

#### If you want a default Cloud Shell session

default.json:

{% gist 998c4fa5a53400acd6e36ab67d71c74d default.json %}

#### If you want a Cloud Shell session connected to a vnet

Isolated-Vnet.json:

{% gist 998c4fa5a53400acd6e36ab67d71c74d Isolated-Vnet.json %}

_Keep in mind, if you use the Isolated Vnet option, the Network Profile, Relay and Storage Profile must already exist._

Then, create run the following Azure CLI commands from a bash shell:

{% gist 998c4fa5a53400acd6e36ab67d71c74d get-bearer-token.sh %}

This command will show you what the current cloud shell settings are for the user:

{% gist 998c4fa5a53400acd6e36ab67d71c74d show-current-user-settings.sh %}

This command will set the cloud shell settings for the user based on the json file you feed into the body parameter:

{% gist 998c4fa5a53400acd6e36ab67d71c74d set-user-settings.sh %}

The **ONLY** drawback to this solution is that unlike most ARM resources, this API call determines which user to update based on the identity of the caller issuing the update command. (i.e., there’s only one ‘/userSettings’ URL which only reads and modifies the settings of the user principal who calls it.) Therefore, User A can’t change user B’s settings for them, each user can only change their own settings - at least as of today. Microsoft doesn’t have anything on the roadmap to address this as of the date of this blog post.

So, while you can use IaC to pre-provision dedicated storage accounts and file shares for each user ahead of time, you'll have to instruct each user to run these Azure CLI commands before they try to open Cloud Shell for the first time. You can also create multiple Isolated-Vnet.json files for each isolated vnet you want to connect to so that the user just has to run the script associated with the vnet they want to connect to when they want to switch.

I hope this is helpful!
