# Autonomi GCP GKE Module

Deploys the Autonomi control plane on GCP using GKE, GCS, and Cloud Load Balancer.

## Features

- VPC network with subnet and secondary ranges (or bring your own)
- Cloud Router + NAT for private node egress
- GKE cluster with autoscaling node pool (or use existing)
- GCS bucket with versioning for checkpoints/audit
- Workload Identity for GCS access
- Google-managed SSL certificate
- Helm release of the Autonomi chart

## Usage

```hcl
module "autonomi" {
  source = "../../modules/gcp"

  project_id      = "my-gcp-project"
  region          = "us-central1"
  cluster_name    = "autonomi-prod"
  machine_type    = "e2-standard-2"
  node_count      = 1
  gcs_bucket_name = "autonomi-prod"
  domain_name     = "autonomi.example.com"

  tags = {
    environment = "production"
  }
}
```

## Bring Your Own Infrastructure

Use an existing GKE cluster:

```hcl
module "autonomi" {
  source         = "../../modules/gcp"
  project_id     = "my-gcp-project"
  create_cluster = false
  cluster_name   = "my-existing-gke"
}
```

## Inputs

| Name | Description | Type | Default |
|------|-------------|------|---------|
| project_id | GCP project ID | string | (required) |
| region | GCP region | string | us-central1 |
| zone | GCP zone | string | us-central1-a |
| network_name | Existing VPC name (empty = create new) | string | "" |
| create_cluster | Create new GKE cluster | bool | true |
| cluster_name | GKE cluster name | string | autonomi |
| machine_type | Node machine type | string | e2-standard-2 |
| node_count | Nodes per zone | number | 1 |
| gcs_bucket_name | GCS bucket name prefix | string | autonomi |
| domain_name | Dashboard domain | string | "" |
| helm_values | Additional Helm values | map(string) | {} |
| tags | Resource labels | map(string) | {managed-by=terraform} |

## Outputs

| Name | Description |
|------|-------------|
| cluster_name | GKE cluster name |
| dashboard_url | Dashboard URL |
| kubeconfig_command | kubectl config command |
| gcs_bucket_name | GCS bucket name |
