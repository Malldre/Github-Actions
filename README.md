# ⚙️ Shared GitHub Actions

Central hub for reusable GitHub Actions workflows. Use these workflows across multiple projects to ensure consistency, reduce duplication, and simplify maintenance.

## 📚 Available Workflows

### 🏗️ Infrastructure & Deployment

**1. Terraform (Modular)**

Three separate workflows for maximum flexibility:

**terraform-plan.yml** - Plan infrastructure changes only

```yaml
uses: Malldre/Github-Actions/.github/workflows/terraform-plan.yml@main
with:
  environment: "PROD"
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
  AWS_REGION: ${{ secrets.AWS_REGION }}
  TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
  TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**terraform-apply.yml** - Apply infrastructure changes (with tag validation)

```yaml
uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
with:
  environment: "PROD"
  download-artifacts: true # Optional: download Lambda/artifacts
  validate-tags: true # Default: true - validates tags before apply
  fail-on-missing-tags: false # Default: false - set true to fail if tags missing
  additional-tags: '{"Project":"MyApp","CostCenter":"Engineering"}' # Optional custom tags
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
  AWS_REGION: ${{ secrets.AWS_REGION }}
  TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
  TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**terraform-destroy.yml** - Destroy infrastructure

```yaml
# Destroy all resources in environment
uses: Malldre/Github-Actions/.github/workflows/terraform-destroy.yml@main
with:
  environment: 'DEV'
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
  AWS_REGION: ${{ secrets.AWS_REGION }}
  TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
  TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}

# Destroy only resources with specific tags
uses: Malldre/Github-Actions/.github/workflows/terraform-destroy.yml@main
with:
  environment: 'DEV'
  destroy-by-tags: '{"Environment":"dev","Type":"temporary"}'
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
  AWS_REGION: ${{ secrets.AWS_REGION }}
  TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
  TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}

# Dry-run (show what would be destroyed)
uses: Malldre/Github-Actions/.github/workflows/terraform-destroy.yml@main
with:
  environment: 'PROD'
  dry-run: true
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
  AWS_REGION: ${{ secrets.AWS_REGION }}
  TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
  TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**terraform.yml** - Orchestrator (optional, calls above workflows based on action)

```yaml
uses: Malldre/Github-Actions/.github/workflows/terraform.yml@main
with:
  environment: "PROD"
  action: "apply" # plan, apply, or destroy
secrets:
  AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
  AWS_REGION: ${{ secrets.AWS_REGION }}
  TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
  TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**2. Docker Build & Push** (`docker-build-push.yml`) - Multi-platform builds, GitHub Container Registry

```yaml
uses: Malldre/Github-Actions/.github/workflows/docker-build-push.yml@main
with:
  image-name: "my-app"
  platforms: "linux/amd64,linux/arm64"
secrets:
  registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### � Package Management

**3. Publish NPM Package** (`publish-package.yml`) - Version checking, testing, GitHub Packages

```yaml
uses: Malldre/Github-Actions/.github/workflows/publish-package.yml@main
secrets:
  NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**4. Semantic Release** (`semantic-release.yml`) - Automated versioning, changelog, git tags

```yaml
uses: Malldre/Github-Actions/.github/workflows/semantic-release.yml@main
secrets:
  NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### �️ Database

**5. Drizzle Migrations** (`drizzle.yml`) - Secure migrations, environment-specific, validation

```yaml
uses: Malldre/Github-Actions/.github/workflows/drizzle.yml@main
with:
  environment: "DEV"
```

_Requires `DATABASE_URL` secret_

### 🔍 Code Quality

**6. Code Quality Check** (`code-quality.yml`) - ESLint, Prettier, tests, security scan

```yaml
uses: Malldre/Github-Actions/.github/workflows/code-quality.yml@main
with:
  run-eslint: true
  run-prettier: true
  run-tests: true
```

**7. Repository Check** (`check-repo.yml`) - Prevents execution in workflow repo, fork detection

### 🏷️ Tag Management

**8. Terraform Tag Validation** (`terraform-tag-validation.yml`) - Validates resources have required tags

```yaml
uses: Malldre/Github-Actions/.github/workflows/terraform-tag-validation.yml@main
with:
  environment: "DEV"
  working-directory: "infra"
  required-tags: "Environment,ManagedBy,Repository,CreatedBy"
  fail-on-missing: false # true to fail, false to warn
```

**9. Terraform Tag Validation** (`terraform-tag-validation.yml`) - Validates tags and generates detailed report

```yaml
uses: Malldre/Github-Actions/.github/workflows/terraform-tag-validation.yml@main
with:
  environment: "DEV"
  working-directory: "infra"
  additional-tags: '{"Project":"MyApp","CostCenter":"Engineering"}'
  fail-on-missing: false # true to fail if tags are missing, false to warn only
```

> **📝 Features:**
>
> - ✅ Validates which resources are missing required tags
> - ✅ Shows exact **line numbers** where tags should be added
> - ✅ Generates **ready-to-use tag blocks** for copy-paste
> - ✅ Supports custom additional tags via JSON
> - ✅ Creates detailed Markdown report + JSON artifact
> - ❌ **Does NOT modify** your code (validation only)

## ⚙️ Setup & Configuration

### 🔐 Required Secrets

Configure these secrets in your repository (Settings → Secrets and variables → Actions):

**⚠️ IMPORTANT:** Secrets must be configured in **each repository that uses these workflows**, not in the Github-Actions repository itself.

#### Option 1: Repository Secrets (Simple)

```
AWS_ROLE_ARN_DEV = arn:aws:iam::652099910794:role/github-actions
AWS_ROLE_ARN_QUAL = arn:aws:iam::ACCOUNT_ID:role/github-actions
AWS_ROLE_ARN_PROD = arn:aws:iam::ACCOUNT_ID:role/github-actions
TF_STATE_BUCKET = your-terraform-state-bucket
TF_LOCK_TABLE = your-terraform-lock-table
DATABASE_URL = postgresql://... (for Drizzle workflows)
```

#### Option 2: Environment Secrets (RECOMMENDED) ⭐

Create environments: `DEV`, `QUAL`, `PROD` (Settings → Environments)

Then add these secrets to EACH environment:

```
AWS_ROLE_ARN = arn:aws:iam::ACCOUNT_ID:role/github-actions
AWS_REGION = us-east-1  (or your preferred region)
TF_STATE_BUCKET = terraform-state-{environment}
TF_LOCK_TABLE = terraform-locks-{environment}
```

**⚠️ IMPORTANT:** All Terraform workflows (`terraform-plan`, `terraform-apply`, `terraform-destroy`) now **require** the `environment` parameter to access environment-scoped secrets.

**Example:**

```yaml
jobs:
  deploy:
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: "DEV" # ← This is REQUIRED to access environment secrets
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }} # From DEV environment
      AWS_REGION: ${{ secrets.AWS_REGION }} # From DEV environment
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }} # From DEV environment
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }} # From DEV environment
```

**Benefits of Environment Secrets:**

- ✅ Automatic protection rules (approvals, wait timers)
- ✅ Same secret name across environments
- ✅ Better separation and security
- ✅ Easier to manage
- ✅ Works seamlessly with all workflows

### 📝 Usage with Environments

```yaml
jobs:
  deploy-dev:
    environment: DEV # Links to 'DEV' environment secrets
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: "DEV"
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }} # From 'DEV' environment
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

## 🚀 Quick Start

**Example 1: Auto-Detect Environment (RECOMMENDED)**

```yaml
name: Deploy Infrastructure
on:
  push:
    branches: [main, homolog, develop]

jobs:
  # Dynamically detect environment based on branch
  setup:
    runs-on: ubuntu-latest
    outputs:
      environment: ${{ steps.set-env.outputs.environment }}
    steps:
      - name: Set Environment
        id: set-env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "environment=PROD" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/homolog" ]]; then
            echo "environment=QUAL" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/develop" ]]; then
            echo "environment=DEV" >> $GITHUB_OUTPUT
          else
            echo "environment=DEV" >> $GITHUB_OUTPUT
          fi

  # Deploy with detected environment
  deploy:
    needs: setup
    environment: ${{ needs.setup.outputs.environment }} # DEV, QUAL, or PROD
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: ${{ needs.setup.outputs.environment }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }} # From environment secret
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**Example 2: Simple Multi-Environment**

```yaml
name: Deploy Infrastructure
on:
  push:
    branches: [main, homolog, develop]

jobs:
  # DEV environment (develop branch)
  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    environment: DEV
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: "DEV"
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}

  # QUAL environment (homolog branch)
  deploy-qual:
    if: github.ref == 'refs/heads/homolog'
    environment: QUAL
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: "QUAL"
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}

  # PROD environment (main branch - with approval)
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    environment: PROD
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: "PROD"
      fail-on-missing-tags: true # Strict validation for production
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**Example 3: Separate Plan and Apply**

```yaml
name: Deploy Infrastructure
on:
  push:
    branches: [main]
  pull_request:

jobs:
  quality:
    uses: Malldre/Github-Actions/.github/workflows/code-quality.yml@main

  # Plan on PRs and main
  terraform-plan:
    needs: quality
    uses: Malldre/Github-Actions/.github/workflows/terraform-plan.yml@main
    with:
      environment: "PROD"
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}

  # Apply only on main (with automatic tag management)
  terraform-apply:
    needs: terraform-plan
    if: github.ref == 'refs/heads/main'
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: "PROD"
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**Example 2: Using Orchestrator (Simpler)**

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  quality:
    uses: Malldre/Github-Actions/.github/workflows/code-quality.yml@main

  deploy:
    needs: quality
    environment: PROD
    uses: Malldre/Github-Actions/.github/workflows/terraform.yml@main
    with:
      environment: "PROD"
      action: "apply" # or 'plan', 'destroy'
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

**Example 3: Custom Tag Management (advanced)**

```yaml
name: Deploy with Custom Tags
on:
  push:
    branches: [main]

jobs:
  deploy:
    environment: PROD
    uses: Malldre/Github-Actions/.github/workflows/terraform-apply.yml@main
    with:
      environment: "PROD"
      # Auto-tagging enabled by default with custom tags
      additional-tags: '{"Project":"MyApp","CostCenter":"Engineering","Owner":"DevTeam"}'
      fail-on-missing-tags: true # Fail if tags still missing after auto-tag
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      TF_STATE_BUCKET: ${{ secrets.TF_STATE_BUCKET }}
      TF_LOCK_TABLE: ${{ secrets.TF_LOCK_TABLE }}
```

## 💡 Key Features

- ✅ **Security**: No credential exposure, OIDC auth, explicit permissions
- ✅ **Performance**: Caching (50-60% faster), timeouts configured
- ✅ **Reliability**: Validations, error handling, detailed summaries
- ✅ **Automatic Tag Management**: Resources are automatically tagged with required tags (Environment, ManagedBy, Repository, CreatedBy)
- ✅ **Tag Validation**: All deployments validate tags before applying changes

## 🏷️ Versioning

```yaml
uses: Malldre/Github-Actions/.github/workflows/terraform.yml@v1  # ✅ Recommended
uses: Malldre/Github-Actions/.github/workflows/terraform.yml@main  # ⚠️ Latest
uses: Malldre/Github-Actions/.github/workflows/terraform.yml@abc123  # ✅ Stable
```

## 🤝 Contributing

Open issues or PRs. Use conventional commits (`feat:`, `fix:`, `docs:`).
