# Autonomi AWS EKS Module

Deploys the Autonomi control plane on AWS using EKS, S3, and ALB.

## Features

- VPC with public/private subnets (or bring your own)
- EKS cluster with managed node group (or use existing)
- S3 bucket with versioning and encryption for checkpoints/audit
- IAM Roles for Service Accounts (IRSA) for S3 access
- AWS ALB Ingress Controller via Helm
- Helm release of the Autonomi chart with S3-aware values
- Security groups for cluster, workers, and ALB

## Usage

```hcl
module "autonomi" {
  source = "../../modules/aws"

  region             = "us-west-2"
  cluster_name       = "autonomi-prod"
  node_instance_type = "t3.large"
  node_count         = 3
  domain_name        = "autonomi.example.com"
  acm_certificate_arn = "arn:aws:acm:us-west-2:123456789:certificate/abc-123"

  tags = {
    Environment = "production"
  }
}
```

## Bring Your Own Infrastructure

Use an existing VPC:

```hcl
module "autonomi" {
  source = "../../modules/aws"
  vpc_id = "vpc-0123456789abcdef0"
}
```

Use an existing EKS cluster:

```hcl
module "autonomi" {
  source         = "../../modules/aws"
  create_cluster = false
  cluster_name   = "my-existing-cluster"
}
```

## Inputs

| Name | Description | Type | Default |
|------|-------------|------|---------|
| region | AWS region | string | us-west-2 |
| vpc_id | Existing VPC ID (empty = create new) | string | "" |
| create_cluster | Create new EKS cluster | bool | true |
| cluster_name | EKS cluster name | string | autonomi |
| cluster_version | Kubernetes version | string | 1.29 |
| node_instance_type | EC2 instance type | string | t3.large |
| node_count | Desired node count | number | 3 |
| s3_bucket_prefix | S3 bucket name prefix | string | autonomi |
| acm_certificate_arn | ACM cert ARN for HTTPS | string | "" |
| domain_name | Dashboard domain | string | "" |
| helm_values | Additional Helm values | map(string) | {} |
| tags | Resource tags | map(string) | {ManagedBy=terraform} |

## Outputs

| Name | Description |
|------|-------------|
| cluster_endpoint | EKS API endpoint |
| cluster_name | EKS cluster name |
| dashboard_url | Dashboard URL |
| kubeconfig_command | kubectl config command |
| s3_bucket_name | S3 bucket name |
| s3_bucket_arn | S3 bucket ARN |
