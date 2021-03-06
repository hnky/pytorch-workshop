# Lab 2 - Create a Azure Machine Learning Workspace

**Welcome to lab 2, in the [previous lab](../Lab%201%20-%20Setup) you have setup your development enviroment, now it is time to setup your Azure Machine Learning workspace. In this workspace you can manage your complete model life cycle.**

To get started we need to setup a few resources in Azure. Please make sure you have an active Azure Subscription and completed [Lab 1](../Lab%201%20-%20Setup).

## Create a resource group

The Azure Machine Learning workspace must be created inside a resource group. You can use an existing resource group or create a new one. To create a new resource group, use the following command. Replace ```<resource-group-name>``` with the name to use for this resource group. Replace ```<location>``` with the Azure region to use for this resource group:

**Example name and location:**&#x20;

* resource group name: pytorchworkshop
* location: WestEurope or eastus

>**Location:** Choose eastus or WestEurope, this is needed for the compute we are using later.

```
az group create --name <resource-group-name> --location <location>
```

## Create the Azure Machine Learning Workspace

To create a new workspace with all the related services, use the following command:

```
az ml workspace create -n <workspace-name> -g <resource-group-name>
```

>If the az ml command does not work run: az extension add -n ml -y

>You can now view your workspace by visiting [https://ml.azure.com](https://ml.azure.com)

## Make things easier by setting defaults

After every az ml command you have to type "-w \<workspace-name> -g \<resource-group-name>". You can make everything a bit easier by settings the values for this parameters by default.

```
az configure --defaults workspace=<workspace-name> group=<resource-group-name>
```

## Create a Compute Cluster

To train our model we need an Azure Machine Learning Compute cluster. To create a new compute cluster, use the following command.

This command will create an Azure Machine Learning Compute cluster with 1 node that is always on and is using Standard_NC4as_T4_v3 virtual Machines.

```
az ml compute create --type amlcompute -n gpu-cluster --min-instances 1 --max-instances 1 --size Standard_NC6
```

> View your created Azure Machine Learning Compute cluster on [https://ml.azure.com](https://ml.azure.com)

> Creating compute can take up to 10 minutes to complete

To see the list of created compute in your workspace you can type:

```
az ml compute list --output table
```

## Review

The setup of your workspace with compute is now completed.    
In this lab you have:

* [ ] Created a resource group
* [ ] Created an Azure Machine Learning Workspace
* [ ] Configured a default resource group and workspace
* [ ] Created Azure Machine Learning Compute Cluster&#x20;
