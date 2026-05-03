# Elephant TF CI

A Go TUI application for creating, managing, and destroying GitHub Actions CI/CD pipelines with AWS OIDC authentication. Built for engineers who want infrastructure automation without the overhead.

---

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [What It Creates](#what-it-creates)
- [Configuration](#configuration)
- [Pipeline Management](#pipeline-management)
- [Security](#security)
- [License](#license)

---

## Features

- **Interactive TUI** — Terminal interface built with Bubble Tea for guided pipeline setup
- **OIDC Authentication** — Keyless AWS authentication using GitHub web identity; no stored credentials
- **GitHub Integration** — Automatic repository and workflow file creation
- **Multi-environment Support** — Works with any branch/environment structure
- **Security Scanning** — Built-in Checkov, TFLint, and TFSec
- **Cost Estimation** — Infracost runs on every PR and posts a cost diff comment before any changes are applied
- **Custom AWS Regions** — Supports any AWS region including GovCloud
- **Pipeline Discovery** — Automatically finds repositories with existing Terraform workflows
- **Real-time Status** — Displays recent workflow runs with status indicators
- **Smart Destroy** — Detects environments from `tfvars` files before destruction, with multi-step confirmation

---

## Installation

### Linux

```bash
curl -L https://github.com/King-Zingelwayo/elephant-tf-ci-release/releases/latest/download/elephant-tf-ci-linux-amd64 -o elephant-tf-ci
chmod +x elephant-tf-ci
sudo mv elephant-tf-ci /usr/local/bin/
```

### macOS

```bash
curl -L https://github.com/King-Zingelwayo/elephant-tf-ci-release/releases/latest/download/elephant-tf-ci-darwin-amd64 -o elephant-tf-ci
chmod +x elephant-tf-ci
sudo mv elephant-tf-ci /usr/local/bin/
```

### Windows

Download `elephant-tf-ci-windows-amd64.exe` from the [releases page](https://github.com/King-Zingelwayo/elephant-tf-ci-release/releases/latest).

### Verify

```bash
elephant-tf-ci
```

---

## Prerequisites

### AWS OIDC Setup

Elephant TF CI uses OIDC for keyless AWS authentication. Complete this setup once before creating your first pipeline.

**Step 1 — Create an OIDC Identity Provider**

```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1 \
  --client-id-list sts.amazonaws.com
```

Alternatively, create it via the AWS Console: IAM → Identity providers → Add provider.

**Step 2 — Create an IAM Role with Web Identity**

In the AWS Console: IAM → Roles → Create role → Web identity. Select `token.actions.githubusercontent.com` as the identity provider and `sts.amazonaws.com` as the audience.

**Step 3 — Configure the Trust Policy**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR-ACCOUNT-ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR-ORG/YOUR-REPO:*"
        }
      }
    }
  ]
}
```

Replace `YOUR-ACCOUNT-ID`, `YOUR-ORG`, and `YOUR-REPO` with your values.

**Step 4 — Attach a Permissions Policy**

Attach the appropriate IAM policies for your Terraform resources (EC2, S3, etc.). Ensure the role has S3 access for the Terraform state bucket.

### Infracost Setup

Elephant TF CI uses [Infracost](https://www.infracost.io) to post a cost breakdown comment on every pull request, showing the estimated monthly cost change before anything is applied.

**Step 1 — Get an API key**

Sign up at [infracost.io](https://www.infracost.io) and retrieve your API key from the dashboard.

**Step 2 — Add the secret to your repository**

The pipeline expects the key as a GitHub Secret named `INFRACOST_API_KEY`. Elephant TF CI stores this automatically during pipeline creation if you provide the key during setup. To add it manually:

```bash
gh secret set INFRACOST_API_KEY --body "your-api-key" --repo YOUR-ORG/YOUR-REPO
```

Once configured, Infracost runs as part of the PR workflow and posts a comment showing the cost diff for the proposed infrastructure changes.

---

## Quick Start

```bash
elephant-tf-ci
```

The application launches an interactive menu. Use arrow keys to navigate and Enter to select.

---

## Usage

### Main Menu

| Option | Description |
|---|---|
| Create New Pipeline | Set up a CI/CD pipeline for a new repository |
| View Existing Pipelines | Manage and monitor existing Terraform workflows |
| Exit | Close the application |

### Pipeline Creation Flow

1. **GitHub Authentication** — One-time OAuth setup; no manual token management required
2. **Repository Selection** — Choose from your accessible repositories and branches
3. **AWS Configuration** — Set the region, S3 state bucket, and IAM role ARN
4. **Repository Settings** — Configure description and visibility
5. **Review & Confirm** — Final summary before any resources are created

### Pipeline Management Options

Once a pipeline is selected, you can:

- Open the repository directly in your browser
- Open GitHub Actions to view workflow runs and logs
- Refresh pipeline status and recent run history
- Destroy environment resources with guided confirmation

---

## What It Creates

### Workflow Files

| File | Purpose |
|---|---|
| `terraform.yml` | Full CI/CD workflow with PR-based plan, cost estimation, and apply |
| `destroy.yml` | Safe resource destruction workflow |

### Secrets and Configuration

- **GitHub Secrets** — AWS region, S3 bucket name, IAM role ARN, and Infracost API key, all encrypted at rest
- **OIDC Authentication** — Keyless AWS access; no long-lived credentials stored
- **Branch Protection** — Environment-specific deployment rules

---

## Configuration

### GitHub Settings

- Authentication via OAuth (no manual token management)
- Repository and branch selection from your accessible resources
- Configurable description and visibility

### AWS Settings

| Setting | Description |
|---|---|
| Region | Any AWS region, including GovCloud |
| S3 State Bucket | Location for Terraform remote state |
| IAM Role ARN | Execution role for the pipeline |
| Security Options | Configure whether security scan failures block the pipeline |

### Pipeline Behavior

- **Pull Requests** — Terraform plan and Infracost cost estimate run automatically; a cost diff comment is posted to the PR; no apply
- **PR merge to `main`/`master`** — Plan and apply to the detected environment
- **PR merge to other branches** — Plan and apply to a branch-specific environment
- **Direct push to branches** — Plan only; no apply
- **`feature/` branches** — Plan only; no apply

---

## Pipeline Management

### Discovery and Monitoring

The management view automatically scans all accessible repositories for Terraform workflows. For each pipeline you can view:

- Recent run status using standard indicators (success, failure, in progress, queued)
- The last 5 workflow runs with timestamps and branch info
- Direct links to the GitHub repository and Actions tab

### Environment Detection

Elephant TF CI determines the target environment using the following logic:

1. Scans for environment variables in `terraform.tfvars`, `variables.tfvars`, `{branch-name}.tfvars`, and `env.tfvars`
2. If an `environment` or `env` variable is found, uses that value for the Terraform state path
3. Falls back to the branch name if no environment variable is present

**Examples:**

```
# tfvars file contains environment = "production"
Environment: production (branch: main)

# No environment variable found
Branch: main
```

### Resource Destruction

Destroying resources follows a multi-step process to prevent accidents:

1. **Environment Selection** — Choose the specific environment to target
2. **Risk Warning** — Clear description of what will be destroyed
3. **Typed Confirmation** — Must type the exact repository and environment name
4. **Final Warning** — Last opportunity to cancel
5. **State Validation** — Checks for actual resources in Terraform state before proceeding
6. **Conditional Execution** — Skips destruction if no resources are found in state
7. **Smart Cleanup** — Deletes the S3 state bucket only after resources have been successfully destroyed

---

## Security

### Authentication

- GitHub access uses OAuth; no long-lived personal access tokens
- AWS access uses OIDC web identity; no static credentials stored in GitHub Secrets
- IAM roles can be scoped per environment or branch

### Scanning and Cost Estimation

The generated workflows include integrated scanning and cost analysis using:

- **Checkov** — Infrastructure policy and compliance checks
- **TFLint** — Terraform-specific linting and best practices
- **TFSec** — Security-focused static analysis for Terraform
- **Infracost** — Cost estimation on every PR; posts a monthly cost diff comment before any changes are applied

Scan failures can be configured to block or warn without blocking, depending on your team's requirements. Infracost always runs in non-blocking mode and posts its output as a PR comment for reviewer awareness.

### Audit Trail

All workflow executions are logged in GitHub Actions and visible in the Actions tab of each repository.

---

## License

MIT License — see the [LICENSE](LICENSE) file for details.
