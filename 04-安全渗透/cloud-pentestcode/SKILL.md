---
name: playbook-cloud
description: Cloud IAM privesc paths and metadata endpoints per provider. Load when the target is cloud or you obtain cloud creds/metadata. Triggers - AWS/GCP/Azure, IAM, S3/blob, 169.254.169.254, access key.
---

# Cloud — Privesc Paths & Metadata

## Metadata endpoints (different per cloud!)
**AWS IMDSv1** (no header): `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/`
**AWS IMDSv2** (requires token):
```
TOKEN=$(curl -X PUT -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" http://169.254.169.254/latest/api/token)
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```
**GCP** (requires header): `curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token`
**Azure** (requires header): `curl -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"`

IMDSv1 vs v2 matters: v1 = any SSRF gives cloud creds. v2 requires PUT first (harder via SSRF).

## AWS IAM privesc — dangerous permission combos
- `iam:CreatePolicyVersion` → attach admin policy to self
- `iam:AttachUserPolicy` → attach AdministratorAccess
- `lambda:CreateFunction` + `iam:PassRole` → execute as privileged role
- `sts:AssumeRole` → pivot to cross-account roles
- `ec2:RunInstances` + `iam:PassRole` → launch instance with admin role
Tools: `pacu` (automated), `pmapper` (permission graph)

## GCP privesc
- Default compute SA often has `editor` (overprivileged)
- `iam.serviceAccountKeys.create` → generate key for any SA
- `cloudfunctions.functions.create` + `iam.serviceAccounts.actAs` → RCE as SA
- Check for `allUsers`/`allAuthenticatedUsers` bindings (public access)

## Azure privesc
- Managed Identity with broad role assignments
- `Contributor` → deploy resources, potentially read Key Vault
- App Registration with high API permissions + admin consent
- Automation account runbooks → execute as automation identity

## What to check everywhere
- Public storage (S3/GCS/Blob) — `--no-sign-request` / anonymous
- Secrets in repos / CI variables / build logs
- User-data scripts (bootstrap credentials)
- MFA not enforced on admin accounts
- Audit logging disabled or misconfigured
