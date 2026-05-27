# Changelog

## v2.0.0 — 2026-05-27

Previous release: v1.0.2

### Added
- added release automation

### Changed
- v2.0.0
- infracost and workspaces
- feat: improving UI navigations
- cleaning
- updated download links
- updated makefile
- codebase

### Fixes
- bug fixed

### Verification
- Built and version-checked all release binaries.

---


All notable changes to this project are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/). Versions follow [Semantic Versioning](https://semver.org/).

---

## v2.0.0 — 2025-07-04

This release is a major feature expansion. It introduces Terraform Cloud-style multi-workspace support, a full Bubble Tea TUI, AI-powered PR review, FinOps and SecOps governance, and a complete pipeline management experience.

### 🚀 Workspaces

- **Multi-workspace support** — setup scans the selected branch for Terraform roots and registers each as a workflow workspace with its own state key, GitHub environment, and IAM role override.
- **Deployment modes** — Simple mode for single-root repos; Landing Zone mode for multi-root DAGs that can span multiple AWS accounts.
- **Per-workspace state isolation** — each workspace gets its own S3 state key (`<workspace>/terraform.tfstate`) via a generated `backend.tf`.
- **Workspace registry metadata** — workspaces carry project, environment, role override, auto-approve, and output artifact metadata.
- **Default workspace selection** — setup asks which workspace appears first and acts as the fallback when no workspace can be inferred from a PR diff.
- **Workspace output capture** — apply runs capture `terraform output -json` and upload it as a named artifact.
- **Workspace environments** — setup creates GitHub environments per workspace and binds apply jobs to them.
- **Workspace environment variables** — workspace metadata and role variables are synced to each GitHub environment automatically.

### 🔗 Workspace Dependencies

- **Dependency configuration** — Landing Zone setup can define upstream/downstream workspace relationships.
- **Upstream state gate** — dependent workspace plans block until upstream `terraform.tfstate` exists in S3.
- **Dependency output injection** — upstream state outputs are exported as `TF_VAR_*` values before plan/apply of dependent workspaces.
- **Dependency-aware PR detection** — PRs touching both an upstream and downstream workspace automatically select the downstream workspace.
- **Multi-workspace PR blocking** — PRs touching multiple independent workspace roots are blocked with a clear error.

### 🤖 AI Review

- **Provider selection** — choose Claude Sonnet, Gemini, or Disabled during setup.
- **Gemini model options** — Gemini 2.5 Pro, 2.5 Flash, 2.0 Flash, or a custom model ID.
- **Contextual review** — AI receives Terraform source, plan JSON, Checkov results, TFSec output, Infracost JSON, and workspace metadata.
- **Evidence-cited findings** — each finding cites whether it was flagged by Checkov, TFSec, or inferred by the AI.
- **Production edge case checks** — VPC Flow Logs, ALB/CloudFront/API Gateway/RDS/EKS logging, network routing, and module-level inputs/outputs.
- **Verdict system** — Approve, Caution, or Block; Block fails the PR check.
- **AI review restricted to PRs** — direct push runs never trigger Claude or Gemini, keeping token usage limited to PR review events.
- **Continuation handling** — larger output budget with automatic continuation when the verdict marker is missing.

### 💰 Infracost

- **Optional cost estimates** — Infracost runs against `plan.json` when `INFRACOST_API_KEY` is set; writes an empty `infracost.json` and continues when not.
- **PR cost section** — PR comments always include an Infrastructure Cost Estimate section, explaining the status when data is unavailable.
- **Cost history artifacts** — merged PR applies upload per-workspace Infracost history artifacts with commit metadata.
- **TUI cost cache** — workspace details can cache and inspect `infracost.json` from review artifacts; Dashboard shows cached monthly cost rollups.
- **Budget thresholds** — governance controls can warn or fail PRs on monthly cost deltas.
- **Cost ownership tags** — tag compliance checks surface in the PR Governance section.

### 🔒 Security and Governance

- **Security scanning** — Checkov, TFLint, and TFSec run on every PR.
- **SARIF upload** — Checkov results can be uploaded for GitHub code scanning annotations.
- **OPA/Conftest** — policy checks run automatically for repos with a `policy/` or `policies/` directory.
- **IAM Access Analyzer** — changed IAM policy documents in Terraform plans are validated.
- **Scheduled drift detection** — generated workflows support scheduled drift runs and S3 state-lock audits.
- **Collapsible Governance section** — PR comments fold every Governance subsection (cost thresholds, ownership tags, OPA, IAM Access Analyzer) individually.
- **Governance settings** — Settings → Governance controls thresholds, required tags, SARIF, OPA, IAM Access Analyzer, secret scanning status, and drift scheduling.

### 🖥️ TUI and Navigation

- **Bubble Tea app shell** — after auth, the app runs in a Bubble Tea root model with shared chrome, screen-stack navigation, status footer, and global keys (`n`, `1`–`4`, `?`, `esc`, `q`).
- **Rich Dashboard** — expandable project groups, cached workspace cost rows, live GitHub Actions status polling, `[n]` Init Wizard, `[g]` DAG view, `[r]` run-all planning.
- **Tabbed detail screens** — project, workspace, and run detail screens use Bubble Tea tab navigation (`tab` / `shift+tab`).
- **Bubble Tea selectors** — Settings, run selection, and workspace selection use reusable Bubble Tea select models.
- **DAG view** — workspaces grouped into dependency waves showing apply order.
- **Selectable run details** — Runs screen opens job/step status, durations, log download, rerun failed jobs, and cancel controls.
- **Live log viewport** — watched logs use a Bubble Tea viewport with polling, follow/pause, scrolling, and completion detection.
- **Job-level log streaming** — prefers GitHub Actions job log endpoints; falls back to run-level log bundles.
- **Help screen** — in-app navigation and workflow safety reference.
- **Split TUI packages** — reusable chrome, tabbed models, DAG resolution, and GitHub log fetching live under `internal/tui` with public adapter packages under `tui/`, `dag/`, and `github/`.

### ⚙️ Pipeline Management

- **Workspace registry editor** — add, edit, delete, reorder, and regenerate workflow metadata for workspaces from Settings.
- **Workspace detail editing** — edit workspace metadata, move to another project group, or delete from the registry.
- **Project group manager** — rename project groups or move workspaces between projects.
- **Project detail screen** — project-focused view with workspace, variable set, team access, and dependency-wave actions.
- **DAG wave dispatch** — dispatch plan runs per dependency wave or across all waves in order.
- **Workspace operations** — dispatch plan, destroy, state-pull, state-version listing, and force-unlock from the workspace detail view.
- **State version inspection** — renders cached S3 state-version metadata including latest marker, version IDs, size, and delete markers.
- **Workspace output inspector** — renders cached Terraform outputs; non-sensitive values can be copied to clipboard.
- **Local artifact cache** — workspace outputs and state-pull artifacts cached into `.outputs/`.
- **Sync status panel** — inspect app config, keychain auth, GitHub environments, and workspace variables for drift.
- **Repository migration** — retarget the managed repository/branch and rerun bootstrap in the new repo.
- **Integration management** — rotate or remove Infracost and AI review provider secrets from Settings.
- **Deleted workspace cleanup** — removed workspace environments are deleted from GitHub when registry changes are pushed.

### 🔑 Variable Sets and Teams

- **Variable sets** — define org, project, and workspace variable sets synced to GitHub environment variables or secrets.
- **Variable set manager** — add, edit, and delete variable sets and individual variables without replacing the full list.
- **Variable resolution view** — shows org/project/workspace resolution with masked sensitive values; non-sensitive values copyable.
- **Stale variable cleanup** — removed entries and variable/secret type changes delete the old GitHub environment key only where it previously applied.
- **Team access manager** — map GitHub team slugs to all projects or a single project; `write`/`admin` teams sync as GitHub environment required reviewers.
- **Team reviewer sync** — GitHub environments updated on each sync so removed or downgraded entries do not linger as stale reviewers.

### 🔧 Workflow Generation

- **YAML workspace variables** — workflows load `workspace-automation/<workspace>.yml` or `automation/<workspace>.yml` into a generated var file, reducing committed `*.tfvars`.
- **Variable source fallback order** — workspace YAML → `*.tfvars` → `variables.tf` defaults.
- **Terraform review artifacts** — PR workflows upload `plan.json`, `tfplan.bin`, `checkov_results.json`, and `infracost.json`.
- **Apply consistency** — apply downloads the saved `tfplan.bin` artifact instead of re-planning.
- **Apply gated to merged PR commits** — push-based apply verifies the commit is from a merged PR before checkout or AWS auth.
- **Dynamic manual-run environments** — workflow dispatch environment options come from configured workspace environments.
- **Manual dispatch safety** — manual dispatches support plan, drift, destroy, state-pull, state-versions, lock-audit, and unlock — never apply.
- **Danger Zone** — keychain wipe, local config reset, and workspace registry reset behind typed confirmations.

### 🐛 Bug Fixes

- **Plan JSON export resilience** — PR review continues with placeholder artifacts when early JSON export fails; the real Terraform error appears in the main plan output.
- **Workspace selection for dependency chains** — PRs touching upstream and downstream workspaces now select the downstream workspace instead of failing as a multi-workspace change.
- **Infracost integration** — Infracost runs against `plan.json` with `infracost diff --format json`; never against a Terraform directory.
- **S3 backend hardening** — generated S3 backend blocks always include `encrypt = true`; bucket setup enables SSE on both plan and apply paths.
- **Terraform plan failure signaling** — workflows fail the check when `terraform plan` errors, after PR comments and review artifacts are prepared.
- **Plan and Infracost output delimiter hardening** — multiline GitHub outputs use unique delimiters; closing delimiter always written on its own line.
- **PR comment failures** — PR comment API errors now fail the workflow instead of being silently swallowed.
- **Infracost PR visibility** — PR comments include an Infrastructure Cost Estimate section even when the formatter step cannot produce normal output.
- **AI review continuation** — larger output budget with continuation request when the verdict marker is missing from the response.
- **Config sync rollback** — shared config save + GitHub sync paths use a rollback helper to prevent partial local-state writes on sync failure.
- **Variable set rollback** — Settings restores the previous variable sets if GitHub environment sync fails.

---

## v1.0.1 — 2025-06-10

### 🔐 Security

- **Removed OAuth client secret from binary** — switched from web OAuth flow to GitHub Device Flow; only the public `GITHUB_CLIENT_ID` is embedded.
- **Deleted `internal/github/client.go`** — removed unused package containing a fake base64 encryption stub for GitHub secrets.
- **S3 state bucket hardening** — bucket creation now enables SSE-AES256 encryption and blocks all public access.

### 🐛 Bug Fixes

- **Branch injection** — selected branch is now correctly injected into `on.push.branches` and `on.pull_request.branches`; custom branch names now work.
- **Apply job trigger** — removed fragile `contains(head_commit.message, 'Merge pull request')` condition that broke with squash/rebase merges.
- **Repository visibility** — `ensureRepository` no longer hardcodes `private: true`; the user's visibility selection is passed through correctly.
- **HCL parsing** — `GetBackendConfig` and `getExistingBucket` use compiled regexes instead of string splitting, preventing incorrect parses on lines with inline comments.
- **Version mismatch in release script** — `--version` output extraction now uses `head -1` to read only the version line.

### 🚀 Improvements

- **Full back navigation** — every wizard step has a `← Back` option; `ErrBack` propagates through the step loop without losing state.
- **Concurrent repository scanning** — `getRepositoriesWithWorkflows` uses a worker pool of 10 goroutines instead of serial API calls.
- **Concurrent org repo fetching** — org repositories fetched in parallel in both `selectRepository` and `getRepositoriesWithWorkflows`.
- **Filtering on all selects** — `.Filtering(true)` added to repository, branch, and pipeline list selects.
- **Recursive navigation replaced with loops** — `showPipelineStatus` and `ViewExistingPipelines` use `for` loops instead of growing the call stack.
- **`buildTreeEntries` helper** — eliminates ~40 lines of duplicated blob creation logic between `updateMultipleFiles` and `createInitialCommit`.
- **Workflow concurrency groups** — both generated workflows have `concurrency:` blocks to prevent parallel runs causing state lock conflicts.
- **Job-level permissions** — `issues: write` and `pull-requests: write` scoped to the `plan` job only; `apply` uses minimal permissions.
- **S3 bucket deletion rewritten** — replaced unreadable Python boto3 one-liner with clean AWS CLI commands in `destroy.yml`.

### 🔧 Code Quality

- **Module renamed** — `indlovu-pipeline` → `github.com/King-Zingelwayo/elephant-tf-ci`; all internal imports updated.
- **Version via ldflags** — `Makefile` is the single source of truth for version; `main.go` defaults to `"dev"` for local builds.
- **Release script hardened** — validates semver format, passes `VERSION` to every `make` target, verifies the binary reports the correct version before publishing, and auto-updates `CHANGELOG.md`.
- **Default version guard** — `make release` without an explicit `VERSION` argument fails with a clear error.
- **Deprecated `mathrand.Seed` removed** — auto-seeded since Go 1.20.
- **Dead code removed** — `getStringValue`, `SetBranch`, `CreateDestroyWorkflowFile`, `setupEnvironments` no-op, `refreshToken` unused field.
- **OAuth server timeout** — device flow HTTP client uses `Timeout: 30s` instead of `http.DefaultClient`.

### 📦 Dependencies

- `github.com/charmbracelet/huh` → `v1.0.0`
- `github.com/charmbracelet/bubbletea` → `v1.3.10`
- `github.com/charmbracelet/bubbles` → `v1.0.0`
- `github.com/charmbracelet/lipgloss` → `v1.1.0`
- `github.com/google/go-github` → `v67` (from `v56`)

---

## v1.0.0 — Initial Release

- Initial release.
