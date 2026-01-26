# Terraform Plan and Apply Examples

Common usage patterns for the reusable-terraform-plan-apply workflow.

## Basic Plan on PR, Apply on Main

Standard GitOps workflow:

```yaml
name: Terraform

on:
  pull_request:
    paths:
      - 'infrastructure/**'
  push:
    branches: [main]
    paths:
      - 'infrastructure/**'

jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure
      apply: ${{ github.ref == 'refs/heads/main' }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## Multiple Environments

Deploy to staging and production with different configurations:

```yaml
name: Terraform

on:
  pull_request:
    paths:
      - 'infrastructure/**'
  push:
    branches: [main]

jobs:
  staging:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure/staging
      environment: staging
      apply: ${{ github.ref == 'refs/heads/main' }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.STAGING_AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}

  production:
    needs: staging
    if: github.ref == 'refs/heads/main'
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure/production
      environment: production
      apply: true
    secrets:
      AWS_ROLE_ARN: ${{ secrets.PRODUCTION_AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## With Same-Approver Validation

Require a different person to approve production deployments:

```yaml
jobs:
  production:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure/production
      environment: production
      apply: true
      allow_same_approver: false
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## With Private Terraform Modules

Load SSH key for accessing private modules:

```yaml
jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure
      load_ssh_key: true
      apply: ${{ github.ref == 'refs/heads/main' }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## Plan-Only (No Apply)

Run plan on all PRs without applying:

```yaml
name: Terraform Plan

on:
  pull_request:
    paths:
      - 'infrastructure/**'

jobs:
  plan:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure
      apply: false
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## With Custom Git Ref

Deploy a specific tag or branch:

```yaml
jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure
      git_ref: v1.2.0
      apply: true
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## With Debug Output

Enable debug logging for troubleshooting:

```yaml
jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure
      debug: true
      apply: false
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## Different AWS Region

Deploy to a different AWS region:

```yaml
jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure
      region: eu-north-1
      apply: true
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## Manual Deployment Workflow

Use workflow_dispatch for manual deployments:

```yaml
name: Manual Terraform Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - staging
          - production
      apply:
        description: 'Apply changes'
        required: true
        type: boolean
        default: false

jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure/${{ inputs.environment }}
      environment: ${{ inputs.environment }}
      apply: ${{ inputs.apply }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## Matrix Strategy for Multiple Directories

Run Terraform for multiple directories in parallel:

```yaml
name: Terraform

on:
  push:
    branches: [main]

jobs:
  terraform:
    strategy:
      matrix:
        directory:
          - infrastructure/networking
          - infrastructure/compute
          - infrastructure/database
      fail-fast: false
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: ${{ matrix.directory }}
      apply: true
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

## Using GitHub Environment Protection

Leverage GitHub environment protection rules:

```yaml
jobs:
  terraform:
    uses: oslokommune/reusable-terraform-plan-apply/.github/workflows/reusable-terraform-plan-apply.yml@v2
    with:
      working_directory: infrastructure/production
      environment: production  # Uses environment protection rules
      apply: true
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AGE_PUBLIC_KEY: ${{ secrets.AGE_PUBLIC_KEY }}
      AGE_SECRET_KEY: ${{ secrets.AGE_SECRET_KEY }}
      GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY: ${{ secrets.GOLDEN_PATH_IAC_PRIVATE_DEPLOY_KEY }}
```

Configure protection rules in GitHub:
1. Go to Settings > Environments > production
2. Add required reviewers
3. Configure deployment branches
4. Set wait timer if needed
