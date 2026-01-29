---
description: Reusable GitHub Actions workflow for Terraform plan and apply operations. Supports OIDC authentication, PR comments, encrypted plan artifacts, and same-approver validation.
user_invocable: true
---

# Terraform Plan and Apply Workflow

This skill provides guidance on using the `reusable-terraform-plan-apply` workflow for running Terraform operations in GitHub Actions.

## When to Use This Skill

Use this skill when:
- Setting up Terraform CI/CD pipelines
- Implementing GitOps workflows for infrastructure
- Configuring secure Terraform plan/apply separation
- Troubleshooting Terraform workflow issues

## Workflow Overview

The workflow provides a secure two-job architecture:

1. **terraform-plan**: Creates and encrypts the execution plan
2. **terraform-apply**: Decrypts and applies the plan (optional)

Key features:
- **OIDC authentication**: Secure AWS access without long-lived credentials
- **PR comments**: Automatic plan output in PR comments
- **Encrypted artifacts**: Plan files encrypted with age for security
- **Same-approver validation**: Prevent self-approval of deployments
- **Plugin caching**: Terraform provider plugin caching for faster runs
- **SSH key loading**: Access private Terraform modules

## Quick Start

```yaml
name: Terraform

on:
  pull_request:
    paths:
      - 'infrastructure/**'
  push:
    branches: [main]

jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure/prod
      apply: ${{ github.ref == 'refs/heads/main' }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## Required Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `AWS_ROLE_ARN` | Yes | AWS role ARN for OIDC authentication |
| `AGE_PUBLIC_KEY` | Yes | Age public key for encrypting plan artifacts |
| `AGE_SECRET_KEY` | Yes | Age secret key for decrypting plan artifacts |
| `GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY` | Yes | SSH key for accessing private Terraform modules |

## Key Features

### Two-Job Architecture

The workflow ensures what is planned is exactly what gets applied:
1. Plan job creates the Terraform plan and encrypts it
2. Apply job downloads, decrypts, and applies the exact same plan
3. No drift possible between plan and apply

### PR Comments

The workflow automatically posts plan output to PRs:
- Shows "In progress" status while running
- Updates with full plan output when complete
- Includes links to workflow logs

### Same-Approver Validation

Set `allow_same_approver: false` to require a different person to approve the deployment than the one who triggered it.

### Encrypted Plan Artifacts

Plan files are encrypted with age before being stored as artifacts, ensuring sensitive infrastructure details are protected.

## Read More

- [reference.md](./reference.md) - Complete input/output reference
- [examples.md](./examples.md) - Common usage patterns
