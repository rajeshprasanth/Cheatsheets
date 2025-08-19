# **Azure Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Azure CLI Configuration](#azure-cli-configuration)
3. [Resource Groups & Management](#resource-groups--management)
4. [Storage Accounts & Blobs](#storage-accounts--blobs)
5. [Virtual Machines](#virtual-machines)
6. [Networking](#networking)
7. [IAM & Role-Based Access](#iam--role-based-access)
8. [App Services & Functions](#app-services--functions)
9. [Other Useful Commands](#other-useful-commands)
10. [Best Practices](#best-practices)

### **Overview**

* `az` CLI manages Azure resources.
* Supports automation and scripting.
* Common services: **VMs, Storage, Networking, Functions**.

### **Azure CLI Configuration**

```bash
# Login to Azure
az login

# Set default subscription
az account set --subscription "MySubscription"

# List subscriptions
az account list --output table

# Show current account
az account show
```

### **Resource Groups & Management**

```bash
# List resource groups
az group list

# Create resource group
az group create --name MyResourceGroup --location eastus

# Delete resource group
az group delete --name MyResourceGroup
```

### **Storage Accounts & Blobs**

```bash
# List storage accounts
az storage account list

# Create storage account
az storage account create --name mystorageacct --resource-group MyResourceGroup --location eastus --sku Standard_LRS

# List blobs in container
az storage blob list --account-name mystorageacct --container-name mycontainer --output table

# Upload blob
az storage blob upload --account-name mystorageacct --container-name mycontainer --name file.txt --file ./file.txt

# Download blob
az storage blob download --account-name mystorageacct --container-name mycontainer --name file.txt --file ./file.txt
```

### **Virtual Machines**

```bash
# List VMs
az vm list --output table

# Create VM
az vm create --resource-group MyResourceGroup --name MyVM --image UbuntuLTS --admin-username azureuser --generate-ssh-keys

# Start VM
az vm start --resource-group MyResourceGroup --name MyVM

# Stop VM
az vm stop --resource-group MyResourceGroup --name MyVM

# Restart VM
az vm restart --resource-group MyResourceGroup --name MyVM
```

### **Networking**

```bash
# List virtual networks
az network vnet list

# Create virtual network
az network vnet create --name MyVNet --resource-group MyResourceGroup --subnet-name MySubnet

# List NSGs
az network nsg list

# Create NSG rule
az network nsg rule create --resource-group MyResourceGroup --nsg-name MyNSG --name AllowSSH --protocol tcp --priority 1000 --destination-port-range 22 --access allow
```

### **IAM & RBAC**

```bash
# List roles
az role definition list

# Assign role
az role assignment create --assignee john.doe@domain.com --role "Contributor" --resource-group MyResourceGroup

# Remove role assignment
az role assignment delete --assignee john.doe@domain.com --role "Contributor" --resource-group MyResourceGroup
```

### **App Services & Functions**

```bash
# List web apps
az webapp list --output table

# Create function app
az functionapp create --resource-group MyResourceGroup --consumption-plan-location eastus --runtime python --functions-version 4 --name MyFuncApp

# Deploy code to function
func azure functionapp publish MyFuncApp
```
