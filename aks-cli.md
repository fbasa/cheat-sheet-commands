# AKS (Azure CLI) Prereqs & context

```bash
# Login + pick subscription
az login
az account set --subscription "<SUB_ID>"

# Make sure you have the AKS commands & kubectl/kubelogin
az extension add --name aks-preview --upgrade
az aks install-cli   # installs kubectl/kubelogin in Cloud Shell; local installs vary
```

# Create clusters (common patterns)

### 1) Basic cluster (system-assigned managed identity + Azure CNI)

```bash
RG="rg-aks-demo"; AKS="aks-demo"; LOC="eastus"
az group create -n $RG -l $LOC
az aks create -g $RG -n $AKS -l $LOC \
  --enable-managed-identity \
  --node-count 3 --node-vm-size Standard_DS2_v2 \
  --network-plugin azure \
  --generate-ssh-keys
```

### 2) With **user-assigned managed identity** (+ enable MI on cluster)

```bash
UAMI_ID="/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<name>"
az aks create -g $RG -n $AKS -l $LOC \
  --enable-managed-identity \
  --assign-identity $UAMI_ID \
  --generate-ssh-keys
# or update an existing cluster to use a UAMI:
az aks update -g $RG -n $AKS --enable-managed-identity --assign-identity $UAMI_ID
```

### 3) **Private cluster**

```bash
az aks create -g $RG -n $AKS -l $LOC \
  --enable-managed-identity \
  --enable-private-cluster \
  --network-plugin azure \
  --generate-ssh-keys
```

Private connectivity tip below in “Access & auth”. Docs (general): ([Microsoft Learn][1])

### 4) With **Key Vault CSI Driver** add-on at create time

```bash
az aks create -g $RG -n $AKS -l $LOC \
  --enable-managed-identity \
  --enable-addons azure-keyvault-secrets-provider \
  --generate-ssh-keys
```

# Access & auth

### Get kubeconfig (merge into `~/.kube/config`)

```bash
az aks get-credentials -g $RG -n $AKS           # cluster user
az aks get-credentials -g $RG -n $AKS --admin   # cluster admin
```

### Use `kubelogin` for Entra ID (AAD) clusters

```bash
# after az aks get-credentials ...
kubelogin convert-kubeconfig -l azurecli
```

### Run commands **against a private AKS** without direct API access

```bash
az aks command invoke -g $RG -n $AKS -c "kubectl get pods -A"
```

Docs: ([Microsoft Learn][5], [Stack Overflow][6])

---

# Inspect & discover

```bash
az aks list -o table
az aks show -g $RG -n $AKS -o jsonc
az aks get-versions -l $LOC -o table
az aks get-upgrades -g $RG -n $AKS -o table
```

# Scale

### Scale default node pool (cluster-level)

```bash
az aks scale -g $RG -n $AKS --node-count 5
```

### Node pool operations

```bash
# Add a new pool
az aks nodepool add -g $RG --cluster-name $AKS --name np1 --node-count 3 --node-vm-size Standard_DS2_v2

# Scale a specific pool
az aks nodepool scale -g $RG --cluster-name $AKS --name np1 --node-count 6

# Update labels/taints
az aks nodepool update -g $RG --cluster-name $AKS --name np1 \
  --labels role=backend env=prod \
  --node-taints "workload=cpu:NoSchedule"

# List / show / delete
az aks nodepool list  -g $RG --cluster-name $AKS -o table
az aks nodepool show  -g $RG --cluster-name $AKS --name np1 -o jsonc
az aks nodepool delete -g $RG --cluster-name $AKS --name np1
```

# Upgrades (control plane, pools, and images)

### See targets

```bash
az aks get-upgrades -g $RG -n $AKS -o table
```

### Upgrade cluster (control plane + all pools by default)

```bash
az aks upgrade -g $RG -n $AKS --kubernetes-version <X.Y.Z> --yes
```

### Upgrade **control plane only** (then pools separately)

```bash
az aks upgrade -g $RG -n $AKS --kubernetes-version <X.Y.Z> --control-plane-only --yes
# then upgrade specific node pool(s):
az aks nodepool upgrade -g $RG --cluster-name $AKS --name np1 --kubernetes-version <X.Y.Z> --yes
```

Practices & flags: soak/drain timeouts, node-by-node rolling behaviors are tunable on nodepools. Docs: ([Microsoft Learn][7])

### **Node image** (OS) update without K8s version bump

```bash
az aks nodepool get-upgrades -g $RG --cluster-name $AKS --name np1 -o table
az aks nodepool upgrade -g $RG --cluster-name $AKS --name np1 --node-image-only --yes
```

# Add-ons

### List available & enabled add-ons

```bash
az aks addon list-available -l $LOC -o table
az aks addon list -g $RG -n $AKS -o table
```

### Enable / disable popular add-ons

```bash
# Azure Key Vault provider for Secrets Store CSI Driver
az aks addon enable  -g $RG -n $AKS --addon azure-keyvault-secrets-provider
az aks addon disable -g $RG -n $AKS --addon azure-keyvault-secrets-provider
```

# Stop / start (save cost on non-prod)

```bash
az aks stop  -g $RG -n $AKS
az aks start -g $RG -n $AKS
# Node pool only (granular):
az aks nodepool stop  -g $RG --cluster-name $AKS --name np1
az aks nodepool start -g $RG --cluster-name $AKS --name np1
```

# Security & identity essentials

### Rotate **cluster certificates**

```bash
az aks rotate-certs -g $RG -n $AKS
```

(Expect a maintenance window; this recreates nodes and can take \~30 min.) Docs: ([Microsoft Learn][11])

### Rotate **OIDC issuer signing keys** (Workload Identity)

```bash
az aks oidc-issuer rotate-signing-keys -g $RG -n $AKS
```

### Update/reset **service principal** credentials (SP-based clusters)

```bash
az aks update-credentials -g $RG -n $AKS \
  --reset-service-principal \
  --service-principal <APP_ID> --client-secret <SECRET>
```

# Useful queries/snippets

```bash
# Grab cluster FQDN, version, and node pool names
az aks show -g $RG -n $AKS --query "{fqdn:fqdn, k8s:currentKubernetesVersion}"
az aks nodepool list -g $RG --cluster-name $AKS --query "[].name" -o tsv

# Wait for long-running ops
az aks wait -g $RG -n $AKS --created   # or --updated/--deleted
```

# Clean up

```bash
az aks delete -g $RG -n $AKS --yes --no-wait
az group delete -n $RG --yes --no-wait
```
