# Azure CLI 

# Global & Session Basics

```bash
az login                               # Sign in via browser
az login --use-device-code             # Use device code (e.g., in SSH)
az login --service-principal -u <appId> -p <secret|@file> --tenant <tenantId>  # SPN login
az account list -o table               # List subscriptions
az account set -s "<sub>"              # Set active subscription
az version                             # Show CLI + ext versions
az upgrade                             # Upgrade Azure CLI
az config set defaults.location=<loc> defaults.group=<rg>  # Set handy defaults
az find "az vm create"                 # Search help/examples
```

# Output, Queries & Flags

```bash
az group list -o table                 # Tabular output
az vm list --query "[].{name:name, rg:resourceGroup}" -o table  # JMESPath query
# Useful flags: --yes (assume yes), --only-show-errors, --debug, --output (json|table|tsv), --query
```

# Resource Groups

```bash
az group create -n <rg> -l <loc>       # Create RG
az group list -o table                 # List RGs
az group delete -n <rg> --yes --no-wait  # Delete RG
```

# Providers & Locations

```bash
az provider list -o table              # Registered providers
az provider register -n Microsoft.KeyVault   # Register a provider
az account list-locations -o table     # Regions/locations
```

# Tagging

```bash
az tag list --resource-id <id>                     # List tags on a resource
az resource tag --ids <id> --tags env=prod owner=me  # Apply/replace tags
az tag create --name costCenter --value 123        # Create subscription-level tag key/value
```

# ARM/Bicep Deployments

```bash
az deployment sub create -f main.bicep -l <loc> -p @params.json  # Subscription-level
az deployment group create -g <rg> -f main.bicep -p @params.json # Resource group level
az deployment group show -g <rg> -n <deploymentName>             # Inspect deployment
```

# Storage (Accounts, Containers, Blobs)

```bash
az storage account create -n <stg> -g <rg> -l <loc> --sku Standard_LRS
az storage container create -n <ctr> --account-name <stg>        # Create container
az storage blob upload --account-name <stg> -c <ctr> -f ./file.txt -n file.txt
az storage blob download --account-name <stg> -c <ctr> -n file.txt -f ./out.txt
az storage blob list --account-name <stg> -c <ctr> -o table
# Tip: auth via: --auth-mode login OR env AZURE_STORAGE_CONNECTION_STRING
```

# Virtual Machines

```bash
az vm image list --offer UbuntuServer --publisher Canonical -o table   # Find images
az vm create -g <rg> -n <vm> --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys
az vm open-port -g <rg> -n <vm> --port 22              # Quick NSG rule
az vm start -g <rg> -n <vm>                            # Start VM
az vm stop -g <rg> -n <vm>                             # Stop (OS shutdown)
az vm deallocate -g <rg> -n <vm>                       # Stop billing for compute
az vm delete -g <rg> -n <vm> --yes
az vm list-ip-addresses -g <rg> -n <vm> -o table       # Get public/private IPs
az vm run-command invoke -g <rg> -n <vm> --command-id RunShellScript --scripts "uname -a"
```

# Networking (VNet, Subnet, NSG, Public IP, NIC)

```bash
az network vnet create -g <rg> -n <vnet> --address-prefix 10.0.0.0/16 \
  --subnet-name <subnet> --subnet-prefix 10.0.1.0/24
az network nsg create -g <rg> -n <nsg>
az network nsg rule create -g <rg> --nsg-name <nsg> -n allow-ssh --priority 1000 \
  --access Allow --protocol Tcp --direction Inbound --source-address-prefixes '*' \
  --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 22
az network public-ip create -g <rg> -n <pip> --sku Standard
az network nic create -g <rg> -n <nic> --vnet-name <vnet> --subnet <subnet> --network-security-group <nsg>
```

# Azure Kubernetes Service (AKS)

```bash
az aks create -g <rg> -n <aks> --node-count 2 --node-vm-size Standard_B4ms --enable-managed-identity
az aks get-credentials -g <rg> -n <aks>               # Merge kubeconfig
az aks nodepool scale -g <rg> --cluster-name <aks> -n nodepool1 --node-count 3
az aks upgrade -g <rg> -n <aks> --kubernetes-version <ver> --control-plane-only
```

# App Service (Web Apps)

```bash
az appservice plan create -g <rg> -n <plan> --sku B1 --is-linux
az webapp create -g <rg> -p <plan> -n <app> --runtime "DOTNET|8.0"
az webapp config appsettings set -g <rg> -n <app> --settings ASPNETCORE_ENVIRONMENT=Production
az webapp deployment source config-zip -g <rg> -n <app> --src ./app.zip
az webapp browse -g <rg> -n <app>
```

# Azure Functions

```bash
az storage account create -n <stg> -g <rg> -l <loc> --sku Standard_LRS
az functionapp create -g <rg> -n <func> -s <stg> -c <loc> --runtime dotnet --functions-version 4
az functionapp config appsettings set -g <rg> -n <func> --settings KEY1=VALUE1
az functionapp restart -g <rg> -n <func>
```

# Azure Container Registry (ACR) & Images

```bash
az acr create -g <rg> -n <acr> --sku Basic
az acr login -n <acr>
az acr build -r <acr> -t <acr>.azurecr.io/myapp:1.0 .
az acr repository list -n <acr> -o table
az acr repository show-tags -n <acr> --repository myapp -o table
```

# Containers (Azure Container Apps / Instances)

```bash
# Container Instances (quick single container)
az container create -g <rg> -n <aci> --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu 1 --memory 1 --ports 80 --dns-name-label <unique-label>
az container logs -g <rg> -n <aci>
az container delete -g <rg> -n <aci> --yes
```

# Key Vault (Secrets & Access)

```bash
az keyvault create -g <rg> -n <kv> -l <loc>
az keyvault secret set --vault-name <kv> -n ConnString --value "Server=...;"
az keyvault secret show --vault-name <kv> -n ConnString --query value -o tsv
az keyvault set-policy -n <kv> --spn <appId> --secret-permissions get list set
```

# Managed Identity & RBAC

```bash
# User-assigned managed identity
az identity create -g <rg> -n <uami>
az identity show -g <rg> -n <uami> -o table

# Role assignments (RBAC)
az role assignment create --assignee <objectId|user@contoso.com> \
  --role "Reader" --scope /subscriptions/<sub>/resourceGroups/<rg>
```

# Logs & Monitoring (quick wins)

```bash
az monitor log-analytics workspace create -g <rg> -n <law> -l <loc>
az monitor diagnostic-settings create --name send-to-law \
  --resource <resourceId> --workspace <lawResourceId> --logs '[{"category":"AuditEvent","enabled":true}]'
```

# Identity & Graph (common lookups)

```bash
az ad signed-in-user show                          # Current user info
az ad sp list --display-name <name> -o table       # Find a service principal (Entra ID)
```

# Extensions

```bash
az extension list -o table
az extension add --name containerapp
az extension update --name aks-preview
```

# Cleanup & Orphans

```bash
az resource list -g <rg> -o table                  # List resources in RG
az resource delete --ids <id>                      # Delete by ID
```
