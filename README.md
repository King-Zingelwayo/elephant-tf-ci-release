# Elephant TF CI

A Go terminal app that creates and manages production-ready Terraform CI/CD pipelines on GitHub Actions — with AWS OIDC, S3 remote state, PR plan comments, security scanning, optional Infracost, and optional AI review.

Built for repositories with a single Terraform root or many Terraform Cloud-style workspaces.

---

## Table of Contents

- [Installation](#installation)
- [AWS Prerequisites](#aws-prerequisites)
- [Quick Start](#quick-start)
- [Pipeline Creation Flow](#pipeline-creation-flow)
- [Workspaces](#workspaces)
- [Workspace Dependencies](#workspace-dependencies)
- [YAML Workspace Variables](#yaml-workspace-variables)
- [GitHub Secrets and Variables](#github-secrets-and-variables)
- [PR Workflow Behavior](#pr-workflow-behavior)
- [AI Review](#ai-review)
- [Infracost](#infracost)
- [Security and Cost Governance](#security-and-cost-governance)
- [Destroy Workflow](#destroy-workflow)
- [Generated Files](#generated-files)
- [Management View](#management-view)
- [Teams and Collaboration](#teams-and-collaboration)
- [Development](#development)
- [Release](#release)
- [License](#license)

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

Download `elephant-tf-ci-windows-amd64.exe` from the [latest release](https://github.com/King-Zingelwayo/elephant-tf-ci-release/releases/latest).

### Verify

```bash
elephant-tf-ci --version
```

---

## AWS Prerequisites

Elephant TF CI requires an AWS IAM role that GitHub Actions can assume via OIDC. No long-lived credentials are used.

**1. Create the GitHub OIDC provider** (once per AWS account):

```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1 \
  --client-id-list sts.amazonaws.com
```

**2. Create an IAM role** with this trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
        }
      }
    }
  ]
}
```

**3. Attach permissions** for your Terraform-managed resources and the S3 state bucket. If using a customer-managed KMS key, also include KMS permissions for that key.

---

## Quick Start

```bash
elephant-tf-ci
```

### Navigation

| Key | Action |
|-----|--------|
| `n` | Init Wizard / Create Project |
| `1` | Dashboard |
| `2` | Projects |
| `3` | Runs |
| `4` | Settings |
| `?` | Help |
| `esc` | Back |
| `q` | Quit |

The app stores its config at `~/.config/elephant-tf-ci/elephant.yml` (Linux) after first setup. GitHub tokens are stored in the OS keychain — never in the config file.

---

## Pipeline Creation Flow

Run `Init Wizard` from the main menu and follow these steps:

1. Authenticate with GitHub via Device Flow (no token pasting required).
2. Select repository and branch.
3. Choose **Simple** (single workspace) or **Landing Zone** (multi-workspace DAG) mode.
4. Review discovered Terraform roots and select which become workspaces.
5. _(Landing Zone)_ Customize workspace project, AWS account, environment, and IAM role.
6. _(Landing Zone)_ Configure workspace dependencies.
7. Set the default workspace for manual runs.
8. Configure AWS region and IAM role ARN.
9. _(Optional)_ Configure variable sets.
10. _(Optional)_ Enable KMS encryption for state.
11. _(Optional)_ Enable Infracost.
12. Choose AI reviewer: **Claude**, **Gemini**, or **Disabled**.
13. Review and confirm — the pipeline is created.

---

## Workspaces

Elephant TF CI scans the selected branch for Terraform root modules and maps each one to a workflow workspace.

Each workspace gets:

- A name derived from its Terraform root path
- A project grouping
- An AWS account profile (defaults to the pipeline account; more can be added later)
- A working directory
- A GitHub environment for apply protection
- Its own S3 state key: `<workspace>/terraform.tfstate`

**Example:**

```text
network-prod  →  infra/network/prod  →  network-prod/terraform.tfstate
app-prod      →  infra/app/prod      →  app-prod/terraform.tfstate
```

---

## Workspace Dependencies

Dependencies mimic Terraform Cloud run ordering.

If `app-prod` depends on `network-prod`:

- A PR touching `app-prod` checks that `network-prod/terraform.tfstate` exists in S3.
- If upstream state is missing, the workflow blocks before plan runs.
- When upstream state exists, its outputs are injected as `TF_VAR_*` values before plan/apply.
- If a PR touches both workspaces, the downstream workspace (`app-prod`) is selected automatically.
- PRs touching multiple **independent** workspace roots are blocked — changes must be split into separate PRs.

**Recommended merge order for a new landing zone:**

```text
1. Open and merge network-prod PR
2. Open and merge app-prod PR
3. Open and merge monitoring-prod PR
```

---

## YAML Workspace Variables

Store non-secret Terraform input variables in repo-managed YAML files instead of committing `*.tfvars`.

**Supported paths:**

```text
workspace-automation/<workspace>.yml
automation/<workspace>.yml
```

**Variable resolution order:**

1. Workspace YAML file (`workspace-automation/` or `automation/`)
2. `*.tfvars` files in the working directory
3. `variables.tf` defaults

**Example `automation/networking.yml`:**

```yaml
variables:
  environment: prod
  vpc_cidr: 10.20.0.0/16
  public_subnet_cidrs:
    - 10.20.1.0/24
    - 10.20.2.0/24
  tags:
    Owner: platform
    CostCenter: shared-network
```

The workflow converts this to `.elephant.generated.auto.tfvars.json` and passes it via `-var-file`. Use Elephant variable sets or GitHub environment secrets for sensitive values.

---

## GitHub Secrets and Variables

### Required

| Name | Type | Description |
|------|------|-------------|
| `AWS_REGION` | Variable | AWS region for the pipeline |
| `PIPELINE_ROLE_ARN` | Secret | IAM role ARN assumed by GitHub Actions |
| `TF_STATE_BUCKET` | Variable | S3 bucket for Terraform state |

### Optional

| Name | Type | Description |
|------|------|-------------|
| `TF_STATE_KMS_KEY_ARN` | Secret | KMS key for state encryption |
| `INFRACOST_API_KEY` | Secret | Enables Infracost cost estimates |
| `AI_REVIEW_PROVIDER` | Variable | `claude`, `gemini`, or empty |
| `ANTHROPIC_API_KEY` | Secret | Required for Claude AI review |
| `GEMINI_API_KEY` | Secret | Required for Gemini AI review |
| `GEMINI_MODEL` | Variable | Gemini model ID (e.g. `gemini-2.5-flash`) |
| `BACKEND_EXISTS` | Variable | Skip S3 bucket creation if `true` |

Per-workspace environment variables (`TERRAFORM_ROLE_ARN`, `TF_WORKSPACE`, `TF_WORKING_DIR`, `TF_DEPENDENCIES`, `TF_OUTPUTS_ARTIFACT`, `ELEPHANT_PROJECT`) are synced automatically by the app.

Variable sets can target the organisation, a project, or a single workspace. Sensitive values are written as GitHub environment secrets and are never stored in the app config.

---

## PR Workflow Behavior

### On pull request open, sync, or reopen

1. Detect the changed Terraform workspace from the diff.
2. Resolve dependency-linked changes to the downstream workspace; block unrelated multi-workspace changes.
3. Block if upstream workspace state is missing.
4. Inject upstream outputs as `TF_VAR_*` values.
5. Load workspace YAML variables into a generated var file.
6. Run `terraform plan` and export `tfplan.bin` and `plan.json`.
7. Run Checkov, TFLint, and TFSec.
8. Run Infracost if configured.
9. Post or update the Terraform PR comment (plan + scan + cost + AI review).
10. Fail the workflow if plan errored (after comments are posted).

### On merge (push to target branch)

Apply only runs when the pushed commit is from a verified merged PR. Direct pushes are skipped entirely.

1. Verify the commit came from a merged PR.
2. Detect the workspace from the push diff.
3. Validate upstream state; block apply if missing.
4. Inject upstream outputs as `TF_VAR_*` values.
5. Download the saved `tfplan.bin` artifact.
6. Run `terraform apply -auto-approve tfplan.bin`.
7. Capture `terraform output -json` and upload as the workspace outputs artifact.

### On direct push

- Runs Terraform plan and logs results.
- Skips PR comments and AI review.

### Manual dispatch

Manual runs never apply infrastructure. Available operations: `plan`, `drift`, `destroy`, `state-pull`, `state-versions`, `lock-audit`, `unlock`.

---

## AI Review

AI review is appended to the existing Terraform PR comment.

### Providers

| Provider | Secret Required |
|----------|----------------|
| Claude Sonnet | `ANTHROPIC_API_KEY` |
| Gemini | `GEMINI_API_KEY` + `GEMINI_MODEL` |
| Disabled | — |

### Gemini model options

- `gemini-2.5-pro`
- `gemini-2.5-flash`
- `gemini-2.0-flash`
- Custom model ID

### What the AI receives

- Terraform source from the workspace directory
- Terraform plan JSON
- Checkov and TFSec results
- Infracost JSON (if available)
- Workspace name, working directory, and dependencies

### Verdicts

| Verdict | Effect |
|---------|--------|
| Approve | PR check passes |
| Caution | PR check passes with warning |
| Block | PR check fails — critical issue found |

Each finding includes an evidence line indicating whether it was flagged by Checkov, TFSec, or inferred by the AI. The review also checks production edge cases scanners commonly miss: VPC Flow Logs, ALB/CloudFront/API Gateway/RDS/EKS logging, network routing, and module-level inputs/outputs.

---

## Infracost

Infracost is optional. Without an API key the workflow writes an empty `infracost.json` and continues.

When enabled:

```bash
infracost diff \
  --path plan.json \
  --format json \
  --out-file infracost.json
```

The PR comment always includes an Infrastructure Cost Estimate section. If Infracost is disabled or produces no data, the section explains that instead of disappearing.

FinOps features:

- PR cost estimate sections
- Per-workspace and project-level monthly cost rollups in the TUI
- Budget thresholds that warn or fail PRs on monthly deltas
- Cost ownership tag compliance checks
- Cost history artifacts retained after apply

---

## Security and Cost Governance

SecOps features:

- Checkov, TFLint, and TFSec on every PR
- SARIF upload for GitHub code scanning annotations
- OPA/Conftest policy checks for repos with a `policy/` or `policies/` directory
- IAM Access Analyzer validation for changed IAM policy documents
- GitHub OIDC — no long-lived AWS credentials
- GitHub environment approvals before apply
- Per-workspace IAM role overrides
- Encrypted S3 state with optional KMS
- Scheduled drift detection and S3 state-lock audit artifacts

Governance settings live under **Settings → Governance**. Threshold and policy checks default to warning unless "fail PRs" is explicitly enabled. In PR comments, the Governance section is collapsed by default and each subsection is individually expandable.

---

## Destroy Workflow

The destroy flow requires multiple confirmations:

1. Select the environment or branch to destroy.
2. Read the destruction warning.
3. Type the environment name to confirm.
4. Confirm one final time.
5. `destroy.yml` is triggered.
6. State is validated before destroy runs.
7. The state bucket is preserved if no resources remain.

---

## Generated Files

Elephant TF CI commits these files to the selected repository branch:

```text
.github/workflows/terraform.yml
.github/workflows/destroy.yml
<workspace-dir>/backend.tf          # one per workspace
```

`terraform.yml` includes workspace and environment dropdowns for manual runs, automatic PR workspace detection, upstream state checks, dependency output injection, YAML var file conversion, S3 bucket setup, plan/scan/cost/AI review steps, artifact sharing between plan and apply jobs, and GitHub environment reviewer sync.

---

## Management View

Open **Projects** to inspect managed repositories and workspaces.

```text
Branch      main
Default     network-prod
Count       3 workspaces
Dependency  2 upstream links
PR Policy   one workspace per PR; upstream state required

* network-prod   infra/network/prod   depends: none
  app-prod       infra/app/prod       depends: network-prod
  monitoring     infra/monitoring     depends: none
```

From the management view you can:

- Open workspace overview, variables, state key, outputs artifact, and dependency details
- Open a DAG view grouped by dependency waves
- Dispatch plan runs per wave or across all waves in order
- Run workspace operations: plan, destroy, state pull, state versions, force unlock
- Edit, move, or delete workspace registry entries
- Cache and inspect workspace outputs, state versions, and Infracost summaries
- View sync status for config, keychain auth, GitHub environments, and workspace variables

The **Runs** screen shows recent GitHub Actions runs across all managed repositories with job/step status, duration, live log streaming, log download, rerun, and cancel controls.

---

## Teams and Collaboration

Configure teams from **Settings** or a project detail screen.

| Role | Effect |
|------|--------|
| `read` | Project metadata visibility in the TUI only |
| `write` | Synced as required reviewer on matching GitHub environments |
| `admin` | Synced as required reviewer on matching GitHub environments |

Teams can be scoped to a single project or applied globally. GitHub enforces approvals at the environment gate — only users in the required reviewer path can approve apply.

---

## Development

```bash
git clone https://github.com/King-Zingelwayo/elephant-tf-ci
cd elephant-tf-ci
go mod tidy
make build
./bin/elephant-tf-ci
```

Build all platforms:

```bash
make build-all
```

Run tests:

```bash
go test ./...
```

---

## Release

```bash
./release.sh          # patch bump
./release.sh minor
./release.sh major
./release.sh v1.2.3
```

The script bumps the version, generates release notes, updates the changelog, builds binaries, verifies `--version`, and publishes through the release repo. Use `ALLOW_DIRTY=1` only when intentionally releasing from a dirty working tree.

---

## License

MIT License. See `LICENSE` for details.

---

Sawubona. Happy building with Elephant TF CI.
