# 🐘 Elephant TF CI - Terraform Pipeline Generator

**Ubuntu-powered CI/CD Pipeline Setup & Management Tool**

Elephant TF CI is a comprehensive Go TUI application that helps you create, manage, and destroy GitHub Actions CI/CD pipelines with AWS OIDC authentication. Built with Ubuntu philosophy - "I am because we are."

## ✨ Features

### 🚀 **Pipeline Creation**
- 🎨 **Interactive TUI** - Beautiful terminal interface using Bubble Tea
- 🔐 **OIDC Authentication** - Secure keyless AWS authentication
- 📁 **GitHub Integration** - Automatic repository and workflow creation
- 🔧 **Multi-Environment** - Support for any branch/environment structure
- 🛡️ **Security Scanning** - Built-in Checkov, TFLint, and TFSec
- 🌐 **Custom AWS Regions** - Support for any AWS region including

### 📊 **Pipeline Management**
- 📋 **Pipeline Discovery** - Automatically finds repositories with Terraform workflows
- 🔄 **Real-time Status** - Shows recent workflow runs with status indicators
- 🌐 **Browser Integration** - Direct links to GitHub repository and Actions
- 📈 **Workflow Monitoring** - View last 5 workflow runs with timestamps and links

### 💥 **Resource Destruction**
- 🧠 **Smart Environment Detection** - Automatically detects environments from tfvars files
- 🌳 **Branch-based Environments** - Uses actual repository branches
- 🛡️ **Triple Confirmation** - Multiple safety checks before destruction
- 🎯 **Targeted Destruction** - Destroy specific environments, not everything
- 🔍 **State Validation** - Checks for actual resources before destroying
- 🗑️ **Smart Bucket Management** - Only deletes S3 bucket when resources are destroyed
- 🔄 **Auto-workflow Updates** - Ensures destroy workflows are always current

### 🌍 **Ubuntu Spirit**
- 🤝 **Community-driven** - Embracing African tech excellence
- 🔓 **Open Source** - Free and accessible to all developers

## 📦 Installation

### Quick Install (Recommended)

**Linux:**
```bash
curl -L https://github.com/King-Zingelwayo/elephant-tf-ci-release/releases/latest/download/elephant-tf-ci-linux-amd64 -o elephant-tf-ci
chmod +x elephant-tf-ci
sudo mv elephant-tf-ci /usr/local/bin/
```

**macOS:**
```bash
curl -L https://github.com/King-Zingelwayo/elephant-tf-ci-release/releases/latest/download/elephant-tf-ci-darwin-amd64 -o elephant-tf-ci
chmod +x elephant-tf-ci
sudo mv elephant-tf-ci /usr/local/bin/
```

**Windows:**
Download `elephant-tf-ci-windows-amd64.exe` from [releases](https://github.com/King-Zingelwayo/elephant-tf-ci-release/releases/latest)


### Verify Installation
```bash
elephant-tf-ci
```

## 🚀 Quick Start

### Prerequisites

1. **AWS OIDC Setup**

   **Step 1: Create OIDC Identity Provider**
   ```bash
   # AWS Console: IAM → Identity providers → Add provider
   # OR AWS CLI:
   aws iam create-open-id-connect-provider \
     --url https://token.actions.githubusercontent.com \
     --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1 \
     --client-id-list sts.amazonaws.com
   ```

   **Step 2: Create IAM Role with Web Identity**
   ```bash
   # AWS Console: IAM → Roles → Create role → Web identity
   # Identity provider: token.actions.githubusercontent.com
   # Audience: sts.amazonaws.com
   ```

   **Step 3: Configure Trust Policy**
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

   **Step 4: Attach Permissions Policy**
   - Attach policies for Terraform operations (EC2, S3, etc.)
   - Ensure S3 access for Terraform state bucket

### Usage

```bash
# Run the application
elephant-tf-ci
```

#### 🎨 **Interactive Menu System**

The application provides a comprehensive menu-driven interface:

**Main Menu Options:**
- 🚀 **Create New Pipeline** - Set up CI/CD for a new repository
- 📋 **View Existing Pipelines** - Manage existing Terraform workflows
- 🚪 **Exit** - Close the application

**Pipeline Creation Flow:**
1. 🔐 **GitHub Authentication** - One-time OAuth setup
2. 📁 **Repository Selection** - Choose from your repositories and branches
3. ☁️ **AWS Configuration** - Set region, S3 bucket, and IAM role
4. 📁 **Repository Settings** - Configure description and visibility
5. 🔍 **Review & Confirm** - Final confirmation before creation

**Pipeline Management Options:**
- 🌐 **Open Repository** - Launch GitHub repository in browser
- 📋 **Open GitHub Actions** - View workflow runs and logs
- 🔄 **Refresh Status** - Update pipeline status and recent runs
- 💥 **Destroy Resources** - Safely destroy environment resources
- ← **Navigation** - Easy back/forward navigation
```

## 🏗️ What It Creates

### 📝 **Workflow Files**
- ✅ **terraform.yml** - Complete CI/CD workflow with PR-based approval
- ✅ **destroy.yml** - Safe resource destruction workflow

### 🔐 **Security & Configuration**
- ✅ **GitHub Secrets** - Encrypted AWS configuration (region, S3 bucket, role ARN)
- ✅ **OIDC Authentication** - Keyless AWS access with web identity
- ✅ **Branch Protection** - Environment-specific deployment rules

### 🛡️ **Built-in Safety Features**
- ✅ **Security Scanning** - Checkov, TFLint, and TFSec integration
- ✅ **Multi-environment Support** - Automatic environment detection
- ✅ **PR-based Approval** - Plan on PRs, apply only on merge
- ✅ **Triple Confirmation** - Multiple safety checks for destruction
- ✅ **Smart tfvars Detection** - Automatic variable file discovery

## 🔧 Configuration

### 🐙 **GitHub Settings**
- **OAuth Authentication** - Secure token-based access (no manual tokens needed)
- **Repository Selection** - Choose from your accessible repositories
- **Branch Selection** - Pick target branch for pipeline deployment
- **Repository Settings** - Description and visibility preferences

### ☁️ **AWS Settings**
- **Region Selection** - Choose from popular regions or enter custom region
- **S3 State Bucket** - Terraform state storage location
- **IAM Role ARN** - Pipeline execution role (contains account ID)
- **Security Options** - Choose to fail or continue on security issues

### 🚀 **Pipeline Features**
- **PR-based Approval** - Plan runs on PRs, apply only after merge
- **Smart Environment Detection** - Automatic detection from tfvars files
- **Branch-based Deployments** - Uses actual repository branch structure
- **Security Scanning** - Checkov, TFLint, TFSec with configurable failure
- **OIDC Authentication** - Keyless AWS access with web identity
- **Flexible tfvars** - Works with or without variable files
- **Feature Branch Protection** - No apply on feature/ branches
- **Custom Regions** - Support for any AWS region including GovCloud

### 📊 **Management Features**
- **Pipeline Discovery** - Automatically finds existing Terraform workflows
- **Status Monitoring** - Real-time workflow run status and history
- **Browser Integration** - Direct links to GitHub repository and Actions
- **Safe Destruction** - Multi-step confirmation for resource cleanup
- **Auto-updates** - Keeps workflow templates current

## 📊 Pipeline Management

### 🔍 **Discovery & Monitoring**
- **Auto-discovery** - Finds all repositories with Terraform workflows
- **Real-time Status** - Shows recent workflow runs with status icons (✅❌🔄⏳)
- **Direct Links** - Click-to-open GitHub repository and Actions pages
- **Workflow History** - View last 5 runs with timestamps and branch info

### 💥 **Resource Destruction**
- **Smart Detection** - Automatically detects environments from tfvars files
- **Branch-based** - Uses actual repository branches when no tfvars environment found
- **Safety First** - Triple confirmation process with typed verification
- **Targeted** - Destroy specific environments, not everything
- **Auto-update** - Ensures destroy workflows use latest templates

### 🧠 **Environment Detection Logic**
```
If tfvars file exists with environment variable:
  → Show "Environment: production (branch: main)"
  → Use environment name for Terraform state path

If no environment variable in tfvars:
  → Show "Branch: main"
  → Use branch name for Terraform state path

If no tfvars file:
  → Show "Branch: main"
  → Use branch name for Terraform state path
```

## 🛡️ Security & Workflow

### Security Features
- **OIDC Authentication** - Keyless AWS access with GitHub web identity
- **No Stored Credentials** - No long-term AWS credentials in secrets
- **Branch-specific Roles** - IAM role restrictions per environment
- **Encrypted Secrets** - All GitHub secrets encrypted at rest
- **Security Scanning** - Integrated Checkov, TFLint, and TFSec
- **Audit Trail** - All actions logged in GitHub Actions

### Approval Workflow
1. **Create PR** → Terraform plan runs automatically
2. **Review PR** → Code and infrastructure changes reviewed together
3. **Approve & Merge** → Single approval gate for both code and infra
4. **Auto Deploy** → Apply runs only on PR merge (not direct push)

### Branch Behavior
- **PRs** → Plan only (shows proposed changes)
- **PR merge to main/master** → Plan + Apply to detected environment
- **PR merge to other branches** → Plan + Apply to branch-specific environment
- **Direct push to branches** → Plan only (no apply)
- **feature/** branches → Plan only (no apply)

### Destruction Safety
1. **Environment Selection** → Choose specific environment to destroy
2. **Risk Warning** → Clear explanation of what will be destroyed
3. **Typed Confirmation** → Must type exact repository/environment name
4. **Final Warning** → Last chance to cancel before destruction
5. **State Validation** → Checks for actual resources in Terraform state
6. **Conditional Execution** → Skips destroy if no resources exist
7. **Smart Cleanup** → Only deletes S3 bucket after successful resource destruction

## 🧠 Smart Environment Detection

Elephant TF CI intelligently detects environments using a sophisticated approach:

### 🔍 **Detection Logic**

1. **Check tfvars Files** - Scans for environment variables in:
   - `terraform.tfvars`
   - `variables.tfvars`
   - `{branch-name}.tfvars`
   - `env.tfvars`

2. **Parse Environment Variables** - Looks for patterns like:
   ```hcl
   environment = "production"
   env = "development"
   ```

3. **Fallback to Branch Names** - Uses branch name if no environment found

### 📊 **Environment Display Examples**

**With tfvars environment:**
```
✅ Environment: production (branch: main)
✅ Environment: development (branch: dev)
```

**Without tfvars environment:**
```
🌳 Branch: main
🌳 Branch: dev
```

## 📊 Pipeline Management Workflow

### 🔍 **View Existing Pipelines**
1. **Auto-discovery** - Scans all accessible repositories
2. **Filter by Workflows** - Shows only repos with Terraform workflows
3. **Select Repository** - Choose from discovered pipelines
4. **Pipeline Status** - View recent runs, links, and management options

### 💥 **Resource Destruction Process**
1. **Environment Selection** - Choose from detected environments/branches
2. **Risk Warning** - Clear explanation of destruction scope
3. **Typed Confirmation** - Must type exact repository/environment name
4. **Final Warning** - Last chance to cancel
5. **AWS Configuration** - Specify region and S3 bucket
6. **State Validation** - Checks if resources exist in Terraform state
7. **Conditional Destroy** - Only destroys if resources are found
8. **Smart Bucket Cleanup** - Deletes S3 bucket only after successful resource destruction

### 🔍 **Smart Destroy Logic**
- **Resource Detection** - Scans Terraform state before attempting destroy
- **Skip Empty States** - Preserves S3 bucket if no resources exist
- **Conditional Cleanup** - Only removes infrastructure when resources are actually destroyed
- **Safety First** - Prevents accidental bucket deletion on empty states

### 🔄 **Workflow Updates**
- **Auto-sync Templates** - Ensures workflows use latest templates
- **Backward Compatibility** - Updates old workflows to new format
- **Template Embedding** - No external dependencies for workflow creation

### 🎆 **Community Impact**
- **Accessibility** - Free and open-source for all developers
- **Education** - Promotes modern DevOps practices
- **Empowerment** - Enables infrastructure automation for everyone
- **Excellence** - Showcases African tech innovation

## 📄 License

MIT License - see LICENSE file for details.

---

**Sawubona!** 🐘 Happy building with Elephant TF CI!