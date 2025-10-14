# Microsoft Azure CLI Cheat Sheet

A comprehensive reference for Azure CLI commands covering essential services for cloud certification and daily operations.

## Table of Contents
- [Configuration & Setup](#configuration--setup)
- [Virtual Machines](#virtual-machines)
- [Storage](#storage)
- [Azure AD (Active Directory)](#azure-ad-active-directory)
- [Virtual Network (VNet)](#virtual-network-vnet)
- [AKS (Azure Kubernetes Service)](#aks-azure-kubernetes-service)
- [Azure Functions](#azure-functions)
- [SQL Database](#sql-database)
- [App Service](#app-service)
- [Key Vault](#key-vault)
- [Azure Monitor](#azure-monitor)
- [Additional Services](#additional-services)

---

## Configuration & Setup

### Login and Configuration
```bash
# Login to Azure
az login

# Login with service principal
az login --service-principal --username APP_ID --password PASSWORD --tenant TENANT_ID

# Login with device code (for headless systems)
az login --use-device-code

# List accounts
az account list

# Show current account
az account show

# Set subscription
az account set --subscription "My Subscription"

# Get subscription ID
az account show --query id --output tsv

# List locations
az account list-locations

# List available resource skus
az vm list-skus --location eastus

# Set default location
az config set defaults.location=eastus

# Set default resource group
az config set defaults.group=myResourceGroup

# View configuration
az config get

# Clear configuration
az config unset defaults.location
```

### CLI Management
```bash
# Check CLI version
az version

# Update CLI
az upgrade

# Install extension
az extension add --name azure-devops

# List extensions
az extension list

# Update extension
az extension update --name azure-devops

# Remove extension
az extension remove --name azure-devops

# Get help
az --help

# Get command help
az vm create --help

# Find commands
az find "virtual machine"
```

---

## Virtual Machines

### VM Management
```bash
# List VMs
az vm list

# List VMs in table format
az vm list --output table

# List VMs in specific resource group
az vm list --resource-group myResourceGroup

# Show VM details
az vm show --resource-group myResourceGroup --name myVM

# Create VM (quick)
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Create VM with specific size
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_DS2_v2 \
  --admin-username azureuser \
  --generate-ssh-keys

# Create Windows VM
az vm create \
  --resource-group myResourceGroup \
  --name myWinVM \
  --image Win2019Datacenter \
  --admin-username azureuser \
  --admin-password MyPassword123!

# Create VM with existing NIC
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --nics myNIC \
  --image UbuntuLTS

# Create VM with public IP
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --public-ip-address myPublicIP \
  --admin-username azureuser \
  --generate-ssh-keys

# Start VM
az vm start --resource-group myResourceGroup --name myVM

# Stop VM (deallocate)
az vm deallocate --resource-group myResourceGroup --name myVM

# Stop VM (without deallocate)
az vm stop --resource-group myResourceGroup --name myVM

# Restart VM
az vm restart --resource-group myResourceGroup --name myVM

# Delete VM
az vm delete --resource-group myResourceGroup --name myVM --yes

# Get VM IP addresses
az vm list-ip-addresses --resource-group myResourceGroup --name myVM

# Get instance view
az vm get-instance-view --resource-group myResourceGroup --name myVM

# Resize VM
az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2

# Convert VM to managed disks
az vm convert --resource-group myResourceGroup --name myVM
```

### VM Images
```bash
# List VM images
az vm image list

# List all images from publisher
az vm image list --publisher Canonical --all

# List popular images
az vm image list --output table

# Show image details
az vm image show --location eastus --publisher Canonical --offer UbuntuServer --sku 18.04-LTS --version latest

# List offers
az vm image list-offers --location eastus --publisher Canonical

# List SKUs
az vm image list-skus --location eastus --publisher Canonical --offer UbuntuServer

# Accept image terms
az vm image accept-terms --publisher microsoft-ads --offer linux-data-science-vm-ubuntu --plan linuxdsvmubuntu
```

### Disks
```bash
# List disks
az disk list

# List disks in resource group
az disk list --resource-group myResourceGroup

# Show disk details
az disk show --resource-group myResourceGroup --name myDisk

# Create empty disk
az disk create \
  --resource-group myResourceGroup \
  --name myDisk \
  --size-gb 128 \
  --sku Standard_LRS

# Create disk from snapshot
az disk create \
  --resource-group myResourceGroup \
  --name myDisk \
  --source mySnapshot

# Attach disk to VM
az vm disk attach \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name myDisk

# Detach disk from VM
az vm disk detach \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name myDisk

# Update disk size
az disk update \
  --resource-group myResourceGroup \
  --name myDisk \
  --size-gb 256

# Delete disk
az disk delete --resource-group myResourceGroup --name myDisk --yes

# Create snapshot
az snapshot create \
  --resource-group myResourceGroup \
  --name mySnapshot \
  --source myDisk

# List snapshots
az snapshot list

# Delete snapshot
az snapshot delete --resource-group myResourceGroup --name mySnapshot --yes
```

### VM Extensions
```bash
# List VM extensions
az vm extension list --resource-group myResourceGroup --vm-name myVM

# Set VM extension
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"fileUris": ["https://example.com/script.sh"],"commandToExecute": "./script.sh"}'

# Delete VM extension
az vm extension delete \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name customScript

# Show extension image
az vm extension image list --location eastus
```

### VM Scale Sets
```bash
# List VM scale sets
az vmss list

# Create VM scale set
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 3 \
  --admin-username azureuser \
  --generate-ssh-keys

# Show scale set details
az vmss show --resource-group myResourceGroup --name myScaleSet

# List instances
az vmss list-instances --resource-group myResourceGroup --name myScaleSet

# Scale out
az vmss scale \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --new-capacity 5

# Update instances
az vmss update-instances \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --instance-ids "*"

# Start instances
az vmss start --resource-group myResourceGroup --name myScaleSet

# Stop instances
az vmss deallocate --resource-group myResourceGroup --name myScaleSet

# Delete scale set
az vmss delete --resource-group myResourceGroup --name myScaleSet
```

### Availability Sets
```bash
# List availability sets
az vm availability-set list

# Create availability set
az vm availability-set create \
  --resource-group myResourceGroup \
  --name myAvailabilitySet \
  --platform-fault-domain-count 2 \
  --platform-update-domain-count 5

# Show availability set
az vm availability-set show \
  --resource-group myResourceGroup \
  --name myAvailabilitySet

# Delete availability set
az vm availability-set delete \
  --resource-group myResourceGroup \
  --name myAvailabilitySet
```

---

## Storage

### Storage Account Management
```bash
# List storage accounts
az storage account list

# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS

# Create storage account with specific replication
az storage account create \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_GRS

# Show storage account
az storage account show \
  --name mystorageaccount \
  --resource-group myResourceGroup

# Update storage account
az storage account update \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --sku Standard_GRS

# Get storage account keys
az storage account keys list \
  --account-name mystorageaccount \
  --resource-group myResourceGroup

# Regenerate storage account key
az storage account keys renew \
  --account-name mystorageaccount \
  --resource-group myResourceGroup \
  --key primary

# Get connection string
az storage account show-connection-string \
  --name mystorageaccount \
  --resource-group myResourceGroup

# Delete storage account
az storage account delete \
  --name mystorageaccount \
  --resource-group myResourceGroup --yes
```

### Blob Storage
```bash
# List containers
az storage container list --account-name mystorageaccount

# Create container
az storage container create \
  --name mycontainer \
  --account-name mystorageaccount

# Delete container
az storage container delete \
  --name mycontainer \
  --account-name mystorageaccount

# List blobs
az storage blob list \
  --container-name mycontainer \
  --account-name mystorageaccount

# Upload blob
az storage blob upload \
  --container-name mycontainer \
  --file ./myfile.txt \
  --name myfile.txt \
  --account-name mystorageaccount

# Upload directory
az storage blob upload-batch \
  --destination mycontainer \
  --source ./myfolder \
  --account-name mystorageaccount

# Download blob
az storage blob download \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./downloaded.txt \
  --account-name mystorageaccount

# Download directory
az storage blob download-batch \
  --destination ./myfolder \
  --source mycontainer \
  --account-name mystorageaccount

# Copy blob
az storage blob copy start \
  --destination-container destcontainer \
  --destination-blob destfile.txt \
  --source-container mycontainer \
  --source-blob myfile.txt \
  --account-name mystorageaccount

# Delete blob
az storage blob delete \
  --container-name mycontainer \
  --name myfile.txt \
  --account-name mystorageaccount

# Generate SAS token
az storage blob generate-sas \
  --container-name mycontainer \
  --name myfile.txt \
  --permissions r \
  --expiry 2024-12-31 \
  --account-name mystorageaccount

# Set blob tier
az storage blob set-tier \
  --container-name mycontainer \
  --name myfile.txt \
  --tier Cool \
  --account-name mystorageaccount
```

### File Shares
```bash
# List file shares
az storage share list --account-name mystorageaccount

# Create file share
az storage share create \
  --name myshare \
  --account-name mystorageaccount

# Delete file share
az storage share delete \
  --name myshare \
  --account-name mystorageaccount

# List files
az storage file list \
  --share-name myshare \
  --account-name mystorageaccount

# Upload file
az storage file upload \
  --share-name myshare \
  --source ./myfile.txt \
  --path myfile.txt \
  --account-name mystorageaccount

# Download file
az storage file download \
  --share-name myshare \
  --path myfile.txt \
  --dest ./downloaded.txt \
  --account-name mystorageaccount

# Delete file
az storage file delete \
  --share-name myshare \
  --path myfile.txt \
  --account-name mystorageaccount
```

### Queue Storage
```bash
# List queues
az storage queue list --account-name mystorageaccount

# Create queue
az storage queue create \
  --name myqueue \
  --account-name mystorageaccount

# Delete queue
az storage queue delete \
  --name myqueue \
  --account-name mystorageaccount
```

### Table Storage
```bash
# List tables
az storage table list --account-name mystorageaccount

# Create table
az storage table create \
  --name mytable \
  --account-name mystorageaccount

# Delete table
az storage table delete \
  --name mytable \
  --account-name mystorageaccount
```

---

## Azure AD (Active Directory)

### User Management
```bash
# List users
az ad user list

# Show user
az ad user show --id user@example.com

# Create user
az ad user create \
  --display-name "John Doe" \
  --password MyPassword123! \
  --user-principal-name john@example.onmicrosoft.com

# Update user
az ad user update \
  --id john@example.onmicrosoft.com \
  --display-name "John Smith"

# Delete user
az ad user delete --id john@example.onmicrosoft.com

# Get signed-in user
az ad signed-in-user show

# List user's group memberships
az ad user get-member-groups --id john@example.onmicrosoft.com
```

### Group Management
```bash
# List groups
az ad group list

# Show group
az ad group show --group myGroup

# Create group
az ad group create \
  --display-name myGroup \
  --mail-nickname myGroup

# Delete group
az ad group delete --group myGroup

# Add member to group
az ad group member add \
  --group myGroup \
  --member-id <user-object-id>

# Remove member from group
az ad group member remove \
  --group myGroup \
  --member-id <user-object-id>

# List group members
az ad group member list --group myGroup
```

### Service Principal Management
```bash
# List service principals
az ad sp list

# Show service principal
az ad sp show --id <app-id>

# Create service principal
az ad sp create-for-rbac --name myServicePrincipal

# Create service principal with role
az ad sp create-for-rbac \
  --name myServicePrincipal \
  --role Contributor \
  --scopes /subscriptions/<subscription-id>/resourceGroups/myResourceGroup

# Reset credentials
az ad sp credential reset --id <app-id>

# Delete service principal
az ad sp delete --id <app-id>
```

### Application Management
```bash
# List applications
az ad app list

# Show application
az ad app show --id <app-id>

# Create application
az ad app create --display-name myApplication

# Update application
az ad app update --id <app-id> --homepage https://example.com

# Delete application
az ad app delete --id <app-id>
```

---

## Virtual Network (VNet)

### VNet Management
```bash
# List virtual networks
az network vnet list

# Create virtual network
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet \
  --address-prefix 10.0.0.0/16

# Create virtual network with subnet
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.1.0/24

# Show virtual network
az network vnet show \
  --resource-group myResourceGroup \
  --name myVNet

# Update virtual network
az network vnet update \
  --resource-group myResourceGroup \
  --name myVNet \
  --dns-servers 8.8.8.8 8.8.4.4

# Delete virtual network
az network vnet delete \
  --resource-group myResourceGroup \
  --name myVNet
```

### Subnet Management
```bash
# List subnets
az network vnet subnet list \
  --resource-group myResourceGroup \
  --vnet-name myVNet

# Create subnet
az network vnet subnet create \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --address-prefix 10.0.2.0/24

# Show subnet
az network vnet subnet show \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet

# Update subnet
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --network-security-group myNSG

# Delete subnet
az network vnet subnet delete \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet
```

### Network Security Groups (NSG)
```bash
# List NSGs
az network nsg list

# Create NSG
az network nsg create \
  --resource-group myResourceGroup \
  --name myNSG

# Show NSG
az network nsg show \
  --resource-group myResourceGroup \
  --name myNSG

# Delete NSG
az network nsg delete \
  --resource-group myResourceGroup \
  --name myNSG

# List NSG rules
az network nsg rule list \
  --resource-group myResourceGroup \
  --nsg-name myNSG

# Create NSG rule (allow SSH)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name allow-ssh \
  --priority 1000 \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# Create NSG rule (allow HTTP)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name allow-http \
  --priority 1001 \
  --destination-port-ranges 80 \
  --access Allow \
  --protocol Tcp

# Update NSG rule
az network nsg rule update \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name allow-ssh \
  --priority 1100

# Delete NSG rule
az network nsg rule delete \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name allow-ssh
```

### Network Interface Cards (NIC)
```bash
# List NICs
az network nic list

# Create NIC
az network nic create \
  --resource-group myResourceGroup \
  --name myNIC \
  --vnet-name myVNet \
  --subnet mySubnet

# Show NIC
az network nic show \
  --resource-group myResourceGroup \
  --name myNIC

# Update NIC
az network nic update \
  --resource-group myResourceGroup \
  --name myNIC \
  --network-security-group myNSG

# Delete NIC
az network nic delete \
  --resource-group myResourceGroup \
  --name myNIC
```

### Public IP Addresses
```bash
# List public IPs
az network public-ip list

# Create public IP
az network public-ip create \
  --resource-group myResourceGroup \
  --name myPublicIP

# Create static public IP
az network public-ip create \
  --resource-group myResourceGroup \
  --name myPublicIP \
  --allocation-method Static

# Show public IP
az network public-ip show \
  --resource-group myResourceGroup \
  --name myPublicIP

# Get IP address
az network public-ip show \
  --resource-group myResourceGroup \
  --name myPublicIP \
  --query ipAddress \
  --output tsv

# Delete public IP
az network public-ip delete \
  --resource-group myResourceGroup \
  --name myPublicIP
```

### Load Balancer
```bash
# List load balancers
az network lb list

# Create load balancer
az network lb create \
  --resource-group myResourceGroup \
  --name myLoadBalancer \
  --sku Standard \
  --vnet-name myVNet \
  --subnet mySubnet \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool

# Show load balancer
az network lb show \
  --resource-group myResourceGroup \
  --name myLoadBalancer

# Create health probe
az network lb probe create \
  --resource-group myResourceGroup \
  --lb-name myLoadBalancer \
  --name myHealthProbe \
  --protocol tcp \
  --port 80

# Create load balancer rule
az network lb rule create \
  --resource-group myResourceGroup \
  --lb-name myLoadBalancer \
  --name myLoadBalancerRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe

# Delete load balancer
az network lb delete \
  --resource-group myResourceGroup \
  --name myLoadBalancer
```

### VPN Gateway
```bash
# Create VPN gateway
az network vnet-gateway create \
  --resource-group myResourceGroup \
  --name myVPNGateway \
  --vnet myVNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1

# List VPN gateways
az network vnet-gateway list

# Show VPN gateway
az network vnet-gateway show \
  --resource-group myResourceGroup \
  --name myVPNGateway

# Delete VPN gateway
az network vnet-gateway delete \
  --resource-group myResourceGroup \
  --name myVPNGateway
```

### VNet Peering
```bash
# Create VNet peering
az network vnet peering create \
  --resource-group myResourceGroup \
  --name myVNet1-to-myVNet2 \
  --vnet-name myVNet1 \
  --remote-vnet myVNet2 \
  --allow-vnet-access

# List VNet peerings
az network vnet peering list \
  --resource-group myResourceGroup \
  --vnet-name myVNet1

# Show VNet peering
az network vnet peering show \
  --resource-group myResourceGroup \
  --name myVNet1-to-myVNet2 \
  --vnet-name myVNet1

# Delete VNet peering
az network vnet peering delete \
  --resource-group myResourceGroup \
  --name myVNet1-to-myVNet2 \
  --vnet-name myVNet1
```

---

## AKS (Azure Kubernetes Service)

### Cluster Management
```bash
# List AKS clusters
az aks list

# Create AKS cluster
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3 \
  --generate-ssh-keys

# Create AKS cluster with specific version
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --kubernetes-version 1.25.6 \
  --node-count 3

# Show AKS cluster
az aks show \
  --resource-group myResourceGroup \
  --name myAKSCluster

# Get credentials
az aks get-credentials \
  --resource-group myResourceGroup \
  --name myAKSCluster

# Get credentials for admin
az aks get-credentials \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --admin

# List available Kubernetes versions
az aks get-versions --location eastus

# Scale cluster
az aks scale \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 5

# Upgrade cluster
az aks upgrade \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --kubernetes-version 1.26.0

# Stop cluster
az aks stop \
  --resource-group myResourceGroup \
  --name myAKSCluster

# Start cluster
az aks start \
  --resource-group myResourceGroup \
  --name myAKSCluster

# Delete cluster
az aks delete \
  --resource-group myResourceGroup \
  --name myAKSCluster --yes
```

### Node Pool Management
```bash
# List node pools
az aks nodepool list \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster

# Add node pool
az aks nodepool add \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name mynodepool \
  --node-count 3

# Show node pool
az aks nodepool show \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name mynodepool

# Scale node pool
az aks nodepool scale \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name mynodepool \
  --node-count 5

# Delete node pool
az aks nodepool delete \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name mynodepool
```

### Add-ons
```bash
# Enable monitoring add-on
az aks enable-addons \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --addons monitoring

# Enable HTTP application routing
az aks enable-addons \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --addons http_application_routing

# Disable add-on
az aks disable-addons \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --addons http_application_routing
```

---

## Azure Functions

### Function App Management
```bash
# List function apps
az functionapp list

# Create function app
az functionapp create \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --storage-account mystorageaccount \
  --consumption-plan-location eastus \
  --runtime node \
  --runtime-version 14 \
  --functions-version 4

# Create function app on Linux
az functionapp create \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --storage-account mystorageaccount \
  --consumption-plan-location eastus \
  --runtime python \
  --runtime-version 3.9 \
  --functions-version 4 \
  --os-type Linux

# Show function app
az functionapp show \
  --resource-group myResourceGroup \
  --name myFunctionApp

# Delete function app
az functionapp delete \
  --resource-group myResourceGroup \
  --name myFunctionApp

# Start function app
az functionapp start \
  --resource-group myResourceGroup \
  --name myFunctionApp

# Stop function app
az functionapp stop \
  --resource-group myResourceGroup \
  --name myFunctionApp

# Restart function app
az functionapp restart \
  --resource-group myResourceGroup \
  --name myFunctionApp
```

### Function App Configuration
```bash
# List app settings
az functionapp config appsettings list \
  --resource-group myResourceGroup \
  --name myFunctionApp

# Set app setting
az functionapp config appsettings set \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --settings KEY1=value1 KEY2=value2

# Delete app setting
az functionapp config appsettings delete \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --setting-names KEY1

# Update connection strings
az functionapp config connection-string set \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --connection-string-type SQLAzure \
  --settings MyConnection="Server=..."
```

### Deployment
```bash
# Deploy from zip
az functionapp deployment source config-zip \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --src app.zip

# Configure continuous deployment
az functionapp deployment source config \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --repo-url https://github.com/user/repo \
  --branch main \
  --manual-integration

# Sync deployment
az functionapp deployment source sync \
  --resource-group myResourceGroup \
  --name myFunctionApp
```

---

## SQL Database

### SQL Server Management
```bash
# List SQL servers
az sql server list

# Create SQL server
az sql server create \
  --resource-group myResourceGroup \
  --name mysqlserver \
  --admin-user myadmin \
  --admin-password MyPassword123!

# Show SQL server
az sql server show \
  --resource-group myResourceGroup \
  --name mysqlserver

# Update SQL server
az sql server update \
  --resource-group myResourceGroup \
  --name mysqlserver \
  --admin-password NewPassword456!

# Delete SQL server
az sql server delete \
  --resource-group myResourceGroup \
  --name mysqlserver --yes
```

### Firewall Rules
```bash
# List firewall rules
az sql server firewall-rule list \
  --resource-group myResourceGroup \
  --server mysqlserver

# Create firewall rule
az sql server firewall-rule create \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name AllowMyIP \
  --start-ip-address 203.0.113.1 \
  --end-ip-address 203.0.113.1

# Allow Azure services
az sql server firewall-rule create \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Delete firewall rule
az sql server firewall-rule delete \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name AllowMyIP
```

### Database Management
```bash
# List databases
az sql db list \
  --resource-group myResourceGroup \
  --server mysqlserver

# Create database
az sql db create \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name myDatabase \
  --service-objective S0

# Show database
az sql db show \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name myDatabase

# Update database
az sql db update \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name myDatabase \
  --service-objective S1

# Delete database
az sql db delete \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name myDatabase --yes

# Export database
az sql db export \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name myDatabase \
  --admin-user myadmin \
  --admin-password MyPassword123! \
  --storage-key-type StorageAccessKey \
  --storage-key <key> \
  --storage-uri https://mystorageaccount.blob.core.windows.net/exports/export.bacpac

# Import database
az sql db import \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name myDatabase \
  --admin-user myadmin \
  --admin-password MyPassword123! \
  --storage-key-type StorageAccessKey \
  --storage-key <key> \
  --storage-uri https://mystorageaccount.blob.core.windows.net/exports/export.bacpac
```

---

## App Service

### App Service Plan Management
```bash
# List app service plans
az appservice plan list

# Create app service plan
az appservice plan create \
  --resource-group myResourceGroup \
  --name myAppServicePlan \
  --sku B1

# Create Linux app service plan
az appservice plan create \
  --resource-group myResourceGroup \
  --name myAppServicePlan \
  --sku B1 \
  --is-linux

# Show app service plan
az appservice plan show \
  --resource-group myResourceGroup \
  --name myAppServicePlan

# Update app service plan
az appservice plan update \
  --resource-group myResourceGroup \
  --name myAppServicePlan \
  --sku S1

# Delete app service plan
az appservice plan delete \
  --resource-group myResourceGroup \
  --name myAppServicePlan --yes
```

### Web App Management
```bash
# List web apps
az webapp list

# Create web app
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myWebApp

# Create web app with runtime
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myWebApp \
  --runtime "NODE|14-lts"

# Show web app
az webapp show \
  --resource-group myResourceGroup \
  --name myWebApp

# Delete web app
az webapp delete \
  --resource-group myResourceGroup \
  --name myWebApp

# Start web app
az webapp start \
  --resource-group myResourceGroup \
  --name myWebApp

# Stop web app
az webapp stop \
  --resource-group myResourceGroup \
  --name myWebApp

# Restart web app
az webapp restart \
  --resource-group myResourceGroup \
  --name myWebApp

# Browse web app
az webapp browse \
  --resource-group myResourceGroup \
  --name myWebApp
```

### Web App Configuration
```bash
# List app settings
az webapp config appsettings list \
  --resource-group myResourceGroup \
  --name myWebApp

# Set app settings
az webapp config appsettings set \
  --resource-group myResourceGroup \
  --name myWebApp \
  --settings KEY1=value1 KEY2=value2

# Delete app setting
az webapp config appsettings delete \
  --resource-group myResourceGroup \
  --name myWebApp \
  --setting-names KEY1

# Configure connection strings
az webapp config connection-string set \
  --resource-group myResourceGroup \
  --name myWebApp \
  --connection-string-type SQLAzure \
  --settings DefaultConnection="Server=..."
```

### Deployment
```bash
# Deploy from zip
az webapp deployment source config-zip \
  --resource-group myResourceGroup \
  --name myWebApp \
  --src app.zip

# Configure Git deployment
az webapp deployment source config \
  --resource-group myResourceGroup \
  --name myWebApp \
  --repo-url https://github.com/user/repo \
  --branch main \
  --manual-integration

# List deployment credentials
az webapp deployment list-publishing-credentials \
  --resource-group myResourceGroup \
  --name myWebApp

# Get deployment logs
az webapp log tail \
  --resource-group myResourceGroup \
  --name myWebApp

# Download logs
az webapp log download \
  --resource-group myResourceGroup \
  --name myWebApp \
  --log-file logs.zip
```

---

## Key Vault

### Key Vault Management
```bash
# List key vaults
az keyvault list

# Create key vault
az keyvault create \
  --resource-group myResourceGroup \
  --name myKeyVault \
  --location eastus

# Show key vault
az keyvault show \
  --name myKeyVault

# Delete key vault
az keyvault delete \
  --name myKeyVault

# Purge deleted key vault
az keyvault purge \
  --name myKeyVault

# Recover deleted key vault
az keyvault recover \
  --name myKeyVault
```

### Secret Management
```bash
# Set secret
az keyvault secret set \
  --vault-name myKeyVault \
  --name mySecret \
  --value "MySecretValue"

# Get secret
az keyvault secret show \
  --vault-name myKeyVault \
  --name mySecret

# Get secret value
az keyvault secret show \
  --vault-name myKeyVault \
  --name mySecret \
  --query value \
  --output tsv

# List secrets
az keyvault secret list \
  --vault-name myKeyVault

# Delete secret
az keyvault secret delete \
  --vault-name myKeyVault \
  --name mySecret

# List deleted secrets
az keyvault secret list-deleted \
  --vault-name myKeyVault

# Recover deleted secret
az keyvault secret recover \
  --vault-name myKeyVault \
  --name mySecret
```

### Key Management
```bash
# Create key
az keyvault key create \
  --vault-name myKeyVault \
  --name myKey \
  --protection software

# List keys
az keyvault key list \
  --vault-name myKeyVault

# Show key
az keyvault key show \
  --vault-name myKeyVault \
  --name myKey

# Delete key
az keyvault key delete \
  --vault-name myKeyVault \
  --name myKey
```

### Certificate Management
```bash
# Create certificate
az keyvault certificate create \
  --vault-name myKeyVault \
  --name myCertificate \
  --policy "$(az keyvault certificate get-default-policy)"

# List certificates
az keyvault certificate list \
  --vault-name myKeyVault

# Show certificate
az keyvault certificate show \
  --vault-name myKeyVault \
  --name myCertificate

# Download certificate
az keyvault certificate download \
  --vault-name myKeyVault \
  --name myCertificate \
  --file certificate.pem

# Delete certificate
az keyvault certificate delete \
  --vault-name myKeyVault \
  --name myCertificate
```

### Access Policies
```bash
# Set access policy
az keyvault set-policy \
  --name myKeyVault \
  --upn user@example.com \
  --secret-permissions get list set delete

# Set access policy for service principal
az keyvault set-policy \
  --name myKeyVault \
  --spn <service-principal-id> \
  --key-permissions get list

# Delete access policy
az keyvault delete-policy \
  --name myKeyVault \
  --upn user@example.com
```

---

## Azure Monitor

### Log Analytics
```bash
# List workspaces
az monitor log-analytics workspace list

# Create workspace
az monitor log-analytics workspace create \
  --resource-group myResourceGroup \
  --workspace-name myWorkspace

# Show workspace
az monitor log-analytics workspace show \
  --resource-group myResourceGroup \
  --workspace-name myWorkspace

# Delete workspace
az monitor log-analytics workspace delete \
  --resource-group myResourceGroup \
  --workspace-name myWorkspace --yes

# Query logs
az monitor log-analytics query \
  --workspace myWorkspaceId \
  --analytics-query "AzureActivity | limit 10"
```

### Metrics
```bash
# List metric definitions
az monitor metrics list-definitions \
  --resource /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM

# Get metrics
az monitor metrics list \
  --resource /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM \
  --metric "Percentage CPU"

# Get metrics with time range
az monitor metrics list \
  --resource /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM \
  --metric "Percentage CPU" \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z
```

### Alerts
```bash
# List alert rules
az monitor metrics alert list

# Create metric alert
az monitor metrics alert create \
  --name myAlert \
  --resource-group myResourceGroup \
  --scopes /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM \
  --condition "avg Percentage CPU > 80" \
  --description "Alert when CPU exceeds 80%"

# Update alert
az monitor metrics alert update \
  --name myAlert \
  --resource-group myResourceGroup \
  --enabled false

# Delete alert
az monitor metrics alert delete \
  --name myAlert \
  --resource-group myResourceGroup
```

### Diagnostic Settings
```bash
# List diagnostic settings
az monitor diagnostic-settings list \
  --resource /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM

# Create diagnostic setting
az monitor diagnostic-settings create \
  --name myDiagnostic \
  --resource /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM \
  --workspace myWorkspaceId \
  --logs '[{"category": "Administrative","enabled": true}]' \
  --metrics '[{"category": "AllMetrics","enabled": true}]'

# Delete diagnostic setting
az monitor diagnostic-settings delete \
  --name myDiagnostic \
  --resource /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM
```

---

## Additional Services

### Resource Groups
```bash
# List resource groups
az group list

# Create resource group
az group create \
  --name myResourceGroup \
  --location eastus

# Show resource group
az group show \
  --name myResourceGroup

# Update resource group tags
az group update \
  --name myResourceGroup \
  --tags Environment=Production Team=Backend

# Delete resource group
az group delete \
  --name myResourceGroup --yes

# Export resource group template
az group export \
  --name myResourceGroup \
  --output-file template.json
```

### Container Instances
```bash
# List container groups
az container list

# Create container
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --dns-name-label mycontainer \
  --ports 80

# Show container
az container show \
  --resource-group myResourceGroup \
  --name mycontainer

# Get logs
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer

# Attach to container
az container attach \
  --resource-group myResourceGroup \
  --name mycontainer

# Delete container
az container delete \
  --resource-group myResourceGroup \
  --name mycontainer --yes
```

### Container Registry
```bash
# List registries
az acr list

# Create registry
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Basic

# Show registry
az acr show \
  --name myregistry

# Login to registry
az acr login --name myregistry

# List repositories
az acr repository list --name myregistry

# List tags
az acr repository show-tags \
  --name myregistry \
  --repository myimage

# Delete registry
az acr delete \
  --name myregistry \
  --resource-group myResourceGroup --yes
```

### Service Bus
```bash
# List namespaces
az servicebus namespace list

# Create namespace
az servicebus namespace create \
  --resource-group myResourceGroup \
  --name myServiceBusNamespace \
  --location eastus

# Create queue
az servicebus queue create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name myQueue

# Create topic
az servicebus topic create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name myTopic

# Create subscription
az servicebus topic subscription create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --topic-name myTopic \
  --name mySubscription
```

### Cosmos DB
```bash
# List accounts
az cosmosdb list

# Create account
az cosmosdb create \
  --resource-group myResourceGroup \
  --name mycosmosdbaccount

# Show account
az cosmosdb show \
  --resource-group myResourceGroup \
  --name mycosmosdbaccount

# List keys
az cosmosdb keys list \
  --resource-group myResourceGroup \
  --name mycosmosdbaccount

# Delete account
az cosmosdb delete \
  --resource-group myResourceGroup \
  --name mycosmosdbaccount --yes
```

---

## Tips and Best Practices

### Query and Filter
```bash
# Use JMESPath queries
az vm list --query "[?storageProfile.osDisk.osType=='Linux'].{Name:name, Location:location}" --output table

# Filter by tag
az resource list --tag Environment=Production

# Get specific value
az vm show --resource-group myResourceGroup --name myVM --query "osProfile.adminUsername" --output tsv
```

### Output Formats
```bash
# JSON output (default)
az vm list --output json

# Table output
az vm list --output table

# TSV output
az vm list --output tsv

# YAML output
az vm list --output yaml
```

### Batch Operations
```bash
# Delete multiple resources
az vm delete --ids $(az vm list --resource-group myResourceGroup --query "[].id" --output tsv) --yes

# Start multiple VMs
az vm start --ids $(az vm list --resource-group myResourceGroup --query "[].id" --output tsv)
```

### Interactive Mode
```bash
# Enter interactive mode
az interactive

# Enable auto-completion
az interactive --style friendly
```

---

This cheat sheet covers the most commonly used Azure CLI commands for certification preparation and daily operations. Practice these commands regularly to build proficiency with Microsoft Azure.
