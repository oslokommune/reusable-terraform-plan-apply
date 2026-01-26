# Terraform Plan and Apply Workflow Reference

Complete reference for all inputs, outputs, and secrets.

## Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `AWS_ROLE_ARN` | Yes | AWS role ARN for OIDC authentication to AWS |
| `GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY` | Yes | SSH deploy key for accessing golden-path-iac Terraform modules |
| `AGE_PUBLIC_KEY` | Yes | Age public key for encrypting the Terraform working directory |
| `AGE_SECRET_KEY` | Yes | Age secret key for decrypting the Terraform working directory |

## Inputs

### Core Configuration

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `working_directory` | string | **Required** | Directory containing Terraform configuration files |
| `environment` | string | - | GitHub deployment environment name |
| `git_ref` | string | `main` | Branch, tag, or SHA to checkout |
| `apply` | boolean | `false` | Whether to apply the Terraform plan |
| `debug` | boolean | `false` | Output debug information |

### AWS Configuration

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `configure_aws_credentials` | boolean | `true` | Configure AWS credentials via OIDC |
| `region` | string | `eu-west-1` | AWS region |
| `role_session_name` | string | `golden-path-iac` | AWS role session name |

### Security Configuration

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `allow_same_approver` | boolean | `true` | Allow the same person who triggered to approve |
| `load_ssh_key` | boolean | `false` | Load SSH key for private module access |

## Outputs

The `terraform-plan` job outputs:

| Output | Description |
|--------|-------------|
| `plan_filename` | Name of the generated plan file |
| `terraform_version` | Terraform version extracted from configuration |
| `exitcode` | Plan exit code (0=no changes, 2=changes pending) |

## Permissions Required

The workflow requires these permissions:
```yaml
permissions:
  id-token: write    # For OIDC authentication
  contents: read     # For checkout
  actions: read      # For artifact download
  pull-requests: write  # For PR comments
```

## Concurrency

The workflow uses concurrency groups to prevent parallel runs:
```yaml
concurrency:
  group: "${{ inputs.environment }}-${{ inputs.working_directory }}"
```

This ensures only one plan/apply runs at a time for each environment + directory combination.

## Plan Exit Codes

Terraform plan uses `--detailed-exitcode`:
- `0`: No changes - terraform apply not needed
- `1`: Error during plan
- `2`: Changes detected - terraform apply may be needed

The apply job only runs when `exitcode == 2` and `apply: true`.

## Artifact Encryption

The workflow encrypts the entire Terraform working directory (including `.terraform/` and the plan file) using the age encryption tool. This ensures:
- Sensitive values in plans are protected
- Provider credentials cached in `.terraform/` are secured
- Artifacts can be safely stored in GitHub

## Terraform Version Detection

The workflow automatically detects the Terraform version from your configuration:
```hcl
terraform {
  required_version = "~> 1.5.0"
}
```

## PR Comment Format

The workflow posts structured comments to PRs:
- Header with working directory name
- Status indicator (in progress, success, failure)
- Collapsible plan output or error details
- Link to workflow logs

Comments are updated in place (not duplicated) using a comment marker.
