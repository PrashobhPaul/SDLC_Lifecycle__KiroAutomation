# State Management

## Backend per environment

```hcl
# envs/<env>/backend.tf
terraform {
  backend "s3" {
    bucket         = "swa-calm-tfstate-${var.aws_account_alias}"
    key            = "envs/${var.environment}/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "swa-calm-tflock-${var.aws_account_alias}"
    encrypt        = true
    kms_key_id     = "alias/swa-calm-tfstate"
  }
}
```

## Per-environment isolation
- One state file per env (BE rule 66 — separate state per environment).
- Workspaces are NOT used — every env has its own backend block.
- DynamoDB lock prevents concurrent applies.

## Drift detection
CI runs `terraform plan` nightly per env; non-empty diff opens a Jira ticket tagged `iac-drift`.

## Sensitive state values
- KMS encryption at rest (S3 SSE-KMS).
- TLS in transit.
- Read access scoped to deployer roles only; no human read access in prod.
- Pre-commit hook blocks committing `.tfstate` or `.tfstate.backup` files.

## State migration
Migrating a resource between modules: use `terraform state mv` in a controlled apply with the squad watching;
never run on production state without G4 approval.

## Disaster recovery
- S3 bucket has versioning + MFA-delete in prod.
- DynamoDB lock table has PITR enabled.
- A read-only replica of the state bucket lives in us-west-2.
