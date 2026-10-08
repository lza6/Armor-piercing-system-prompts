# Autonomi Azure AKS Module

Deploys the Autonomi control plane on Azure using AKS and Blob Storage.

## Features

- Resource group for all resources
- VNet with subnet (or bring your own)
- AKS cluster with autoscaling node pool (or use existing)
- Azure Blob Storage with versioning for checkpoints/audit
- System-assigned managed identity with storage access
- Application Gateway ingress via annotations
- Helm release of the Autonomi chart

## Usage

```hcl
module "autonomi" {
  source = "../../modules/azure"

  location             = "eastus"
  resource_group_name  = "autonomi-prod-rg"
  cluster_name         = "autonomi-prod"
  node_vm_size         = "Standard_D2s_v3"
  node_count           = 3
  storage_account_name = "autonomiprod"
  domain_name          = "autonomi.example.com"

  tags = {
    Environment = "production"
  }
}
```

## Bring Your Own Infrastructure

Use an existing AKS cluster:

```hcl
module "autonomi" {
  source              = "../../modules/azure"
  create_cluster      = false
  cluster_name        = "my-existing-aks"
  resource_group_name = "my-rg"
}
```

## Inputs

| Name | Description | Type | Default |
|------|-------------|------|---------|
| location | Azure region | string | eastus |
| resource_group_name | Resource group name | string | autonomi-rg |
| vnet_id | Existing VNet ID (empty = create new) | string | "" |
| create_cluster | Create new AKS cluster | bool | true |
| cluster_name | AKS cluster name | string | autonomi |
| node_vm_size | VM size | string | Standard_D2s_v3 |
| node_count | Node count | number | 3 |
| storage_account_name | Storage account name prefix | string | autonomistorage |
| domain_name | Dashboard domain | string | "" |
| helm_values | Additional Helm values | map(string) | {} |
| tags | Resource tags | map(string) | {ManagedBy=terraform} |

## Outputs

| Name | Description |
|------|-------------|
| cluster_name | AKS cluster name |
| resource_group | Resource group name |
| dashboard_url | Dashboard URL |
| kubeconfig_command | kubectl config command |
| storage_account_name | Storage account name |
| storage_container_name | Blob container name |
