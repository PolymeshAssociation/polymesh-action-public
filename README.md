# SSH Commit Auth & Fast-Forward Merge Action

A comprehensive GitHub Action that combines SSH commit signature authentication with fast-forward merge capabilities, providing enterprise-grade security and user-friendly automation.

## 🔒 Features

- **SSH Commit Authentication**: Validates all commits are signed with approved SSH keys
- **Fast-Forward Merge**: Automated merge operations with conflict detection and resolution guidance
- **Enhanced Security**: Comprehensive input validation, secure file handling, and audit logging
- **User-Friendly**: Clear error messages with step-by-step resolution guidance
- **Production Ready**: Retry logic, rate limiting, and comprehensive error handling
- **Self-Contained**: No third-party action dependencies for maximum security and reliability

## 🚀 Quick Start

### 1. Basic Authentication Only (PR Events)

```yaml
name: Commit Authentication
on:
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  validate-commits:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      issues: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for commit history validation

      - uses: PolymeshAssociation/polymesh-action-public@v1
        with:
          allowed-signers: ${{ vars.MIDDLEWARE_ALLOWED_SIGNERS }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          comment-mode: 'on-error'
```

### 2. Combined Authentication and Fast-Forward

```yaml
name: Auth and Fast-Forward
on:
  pull_request:
    types: [opened, reopened, synchronize]
  issue_comment:
    types: [created]

jobs:
  auth-and-merge:
    if: >
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       contains(github.event.comment.body, '/fast-forward') &&
       github.event.issue.pull_request)
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for commit history validation

      - uses: PolymeshAssociation/polymesh-action-public@v1
        with:
          allowed-signers: ${{ vars.MIDDLEWARE_ALLOWED_SIGNERS }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          comment-mode: 'always'
          auth-required: 'true'
```

## 📋 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `allowed-signers` | SSH allowed signers file content | ✅ | - |
| `github-token` | GitHub token with appropriate permissions | ✅ | - |
| `comment-mode` | Comment feedback level: `always`, `on-error`, `never` | ❌ | `on-error` |
| `merge-method` | Merge method: `fast-forward`, `merge` | ❌ | `fast-forward` |
| `required-key-type` | Required SSH key type (e.g., `ed25519-sk`, `rsa`, `ecdsa`) | ❌ | - |
| `auth-required` | Enforce authentication before fast-forward | ❌ | `true` |
| `base-branch` | Base branch for authentication validation | ❌ | - |
| `head-branch` | Head branch for authentication validation | ❌ | - |

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `auth-status` | Authentication result: `success`, `failed`, `skipped` |
| `merge-status` | Merge result: `success`, `failed`, `skipped`, `blocked` |
| `failed-commits` | JSON array of failed commit information |
| `merge-sha` | SHA of the merge commit if successful |

## 🔧 Usage Scenarios

### Scenario 1: Authentication on Pull Requests

Automatically validate all commits when PRs are opened or updated:

```yaml
on:
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  auth:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for commit history validation

      - uses: PolymeshAssociation/polymesh-action-public@v1
        with:
          allowed-signers: ${{ vars.ALLOWED_SIGNERS }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Scenario 2: Fast-Forward via Comment

Allow authorized users to trigger fast-forward merges via `/fast-forward` comment:

```yaml
on:
  issue_comment:
    types: [created]

jobs:
  fast-forward:
    if: >
      contains(github.event.comment.body, '/fast-forward') &&
      github.event.issue.pull_request
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for commit history validation

      - uses: PolymeshAssociation/polymesh-action-public@v1
        with:
          allowed-signers: ${{ vars.ALLOWED_SIGNERS }}
          github-token: ${{ secrets.MERGE_TOKEN }}
          auth-required: 'true'
```

### Scenario 3: Manual Workflow Dispatch

Enable manual triggering with custom branch specification:

```yaml
on:
  workflow_dispatch:
    inputs:
      base_branch:
        description: 'Base branch'
        required: true
        default: 'main'
      head_branch:
        description: 'Head branch'
        required: true

jobs:
  manual-merge:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for commit history validation

      - uses: PolymeshAssociation/polymesh-action-public@v1
        with:
          allowed-signers: ${{ vars.ALLOWED_SIGNERS }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          base-branch: ${{ github.event.inputs.base_branch }}
          head-branch: ${{ github.event.inputs.head_branch }}
```

## 🛡️ Security Features

### Input Validation
- Comprehensive validation of all inputs with security checks
- Protection against injection attacks and malicious content
- Secure temporary file handling with restricted permissions
- Automatic masking of sensitive information in logs

### SSH Authentication
- Validates SSH commit signatures against allowed signers list
- Supports multiple key formats (RSA, ECDSA, Ed25519, Ed25519-SK)
- Optional key type restrictions for enhanced security
- Detailed failure reporting with resolution guidance

### Audit & Monitoring
- Comprehensive security event logging
- Operation tracking with detailed audit trails
- Rate limiting and abuse prevention
- Automatic alerting for high-severity security events

## 🔍 Error Handling & Troubleshooting

### Action Setup Issues

**Issue**: `Failed to get commit range: The process '/usr/bin/git' failed with exit code 128`

This error occurs when the repository is not checked out before the action runs. **Solution:**

```yaml
steps:
  # ✅ REQUIRED: Always checkout the repository first
  - name: Checkout repository
    uses: actions/checkout@v4
    with:
      fetch-depth: 0  # Required for commit history validation

  # ✅ Then run the action
  - uses: PolymeshAssociation/polymesh-action-public@v1
    with:
      allowed-signers: ${{ vars.ALLOWED_SIGNERS }}
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

**Why this is required**: The action needs access to the git repository and full commit history to validate signatures. Without `actions/checkout@v4` with `fetch-depth: 0`, the runner doesn't have the necessary git data.

### Common SSH Authentication Issues

**Issue**: Commits not signed with SSH keys
```bash
# Configure SSH commit signing
git config --global commit.gpgsign true
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/your_signing_key
```

**Issue**: SSH key not in allowed signers list
- Verify your SSH public key is included in the allowed signers configuration
- Ensure the key format matches the expected format in the signers file

### Fast-Forward Merge Issues

**Issue**: Fast-forward not possible (conflicts detected)
```bash
# Resolve with rebase
git fetch origin
git rebase origin/main  # Replace 'main' with your base branch
git push --force-with-lease
```

**Issue**: Branch protection rules preventing merge
- Ensure all required status checks have passed
- Verify sufficient approving reviews are in place
- Check that the GitHub token has appropriate permissions

## 📊 Monitoring & Analytics

The action provides detailed metrics and audit logs for compliance and monitoring:

- **Authentication Metrics**: Success rates, failure patterns, key type usage
- **Merge Metrics**: Fast-forward success rates, conflict frequency, resolution times
- **Security Events**: Permission violations, suspicious activity, rate limiting
- **Performance Metrics**: Execution times, API call patterns, resource usage

## 🔄 Migration from Existing Workflows

### From `authenticate-commits.yml`

Replace your existing authentication workflow:

```diff
- uses: some-third-party/ssh-auth@v1
+ uses: PolymeshAssociation/polymesh-action-public@v1
  with:
    allowed-signers: ${{ vars.MIDDLEWARE_ALLOWED_SIGNERS }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### From `fast-forward.yml`

Replace fast-forward actions with integrated solution:

```diff
- uses: sequoia-pgp/fast-forward@v1
+ uses: PolymeshAssociation/polymesh-action-public@v1
  with:
    allowed-signers: ${{ vars.MIDDLEWARE_ALLOWED_SIGNERS }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    auth-required: 'true'
```

## 🏷️ Version Pinning

This action follows [Semantic Versioning](https://semver.org/). Users can reference the action using:

- **Specific versions**: `@v1.0.3` (recommended for maximum stability)
- **Minor versions**: `@v1.0` (automatically receives bug fixes)
- **Major versions**: `@v1` (automatically receives compatible updates)
- **Commit SHAs**: `@abc123...` (maximum security and immutability)

### Version Reference Best Practices

For **production workflows**, use specific version tags for stability:
```yaml
- uses: PolymeshAssociation/polymesh-action-public@v1.0.3
```

For **development workflows**, use major version tags to receive updates:
```yaml
- uses: PolymeshAssociation/polymesh-action-public@v1
```

For **maximum security**, use commit SHAs (immutable):
```yaml
- uses: PolymeshAssociation/polymesh-action-public@abc123def456...
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- 🐛 [Report Issues](https://github.com/PolymeshAssociation/polymesh-action-public/issues)
- 💬 For general support, please contact Polymesh Labs

## 📦 Release Information

This is a distribution-only repository. The action is built and published automatically from a private source repository.

**Current Version**: 1.0.3

See the [Releases](https://github.com/PolymeshAssociation/polymesh-action-public/releases) page for version history and changelogs.
