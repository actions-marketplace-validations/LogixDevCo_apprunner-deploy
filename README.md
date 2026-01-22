# AWS App Runner Deploy Action

A GitHub Action that deploys applications to AWS App Runner from Pull Requests, branches, or tags with automatic Docker image building and deployment tracking.

## Features

- 🚀 Deploy from PRs, branches, or tags
- 🐳 Automatic Docker image building with BuildKit
- 📦 ECR integration with caching
- 🏷️ Deployment tracking with GitHub Deployments
- 🔔 Sentry release integration
- 📊 Slack notifications
- ⚡ Simpler than ECS - fully managed

## Usage

### Prerequisites

**Required in your workflow:**
1. Checkout your repository with fetch-depth: 0
2. Configure AWS credentials

### Deploy from Branch

```yaml
name: Deploy to App Runner
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-24.04
    permissions:
      id-token: write
      contents: read
      deployments: write
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
        with:
          fetch-depth: 0
      
      - uses: aws-actions/configure-aws-credentials@61815dcd50bd041e203e49132bacad1fd04d2708
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1
      
      - name: Deploy to App Runner
        uses: LogixDevCo/apprunner-deploy-action@v1.0.0
        with:
          deploy-type: 'from-branch'
          branch: 'main'
          environment: 'staging'
          ecr-repository: 'my-app-staging'
          target-url: 'https://staging.example.com'
```

### Deploy from Pull Request

```yaml
name: Deploy PR to App Runner
on:
  workflow_dispatch:
    inputs:
      pull-request-nb:
        description: 'PR number'
        required: true

jobs:
  deploy:
    runs-on: ubuntu-24.04
    permissions:
      id-token: write
      contents: read
      pull-requests: read
      deployments: write
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
        with:
          ref: main
          fetch-depth: 0
      
      - uses: aws-actions/configure-aws-credentials@61815dcd50bd041e203e49132bacad1fd04d2708
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1
      
      - name: Deploy to App Runner
        uses: LogixDevCo/apprunner-deploy-action@v1.0.0
        with:
          deploy-type: 'from-pr'
          pull-request-nb: ${{ github.event.inputs.pull-request-nb }}
          environment: 'staging'
          ecr-repository: 'my-app-staging'
          target-url: 'https://staging.example.com'
```

### With Sentry and Slack

```yaml
- name: Deploy to App Runner
  uses: LogixDevCo/apprunner-deploy-action@v1.0.0
  with:
    deploy-type: 'from-tag'
    commit-tag: ${{ github.ref_name }}
    environment: 'production'
    ecr-repository: 'my-app-production'
    target-url: 'https://example.com'
    sentry-project: 'my-app'
    sentry-org: 'my-org'
    sentry-environment: 'production'
    sentry-token: ${{ secrets.SENTRY_AUTH_TOKEN }}
    slack-webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `deploy-type` | Deployment type (from-branch, from-pr, from-tag) | Yes | - |
| `environment` | Target environment | Yes | - |
| `ecr-repository` | ECR repository name | Yes | - |
| `target-url` | Target URL of environment | Yes | - |
| `branch` | Branch to deploy (for from-branch) | No | - |
| `pull-request-nb` | PR number (for from-pr) | No | - |
| `commit-tag` | Tag to deploy (for from-tag) | No | - |
| `sentry-project` | Sentry project name | No | `''` |
| `sentry-org` | Sentry organization | No | `''` |
| `sentry-environment` | Sentry environment | No | `''` |
| `sentry-token` | Sentry auth token | No | `''` |
| `slack-webhook` | Slack webhook URL | No | `''` |

## Workflow Steps

1. **Set Deployment Ref**
2. **Fetch PR Details** (if from-pr)
3. **Merge PR** (if from-pr)
4. **Start Deployment Tracking**
5. **Login to ECR**
6. **Build Docker Image** with BuildKit caching
7. **Push to ECR** (triggers App Runner auto-deploy)
8. **Update Deployment Status**
9. **Create Sentry Release** (if configured)
10. **Send Slack Notification** (if configured)

## App Runner Auto-Deploy

App Runner automatically deploys when:
- New image is pushed to ECR with `latest` tag
- App Runner service is configured with automatic deployment

Configure your App Runner service:
```bash
aws apprunner create-service \
  --service-name my-app \
  --source-configuration '{
    "ImageRepository": {
      "ImageIdentifier": "123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
      "ImageRepositoryType": "ECR",
      "ImageConfiguration": {
        "Port": "8080"
      }
    },
    "AutoDeploymentsEnabled": true
  }'
```

## Required Permissions

```yaml
permissions:
  id-token: write      # For AWS OIDC
  contents: read       # For checkout
  pull-requests: read  # For PR details
  deployments: write   # For deployment tracking
```

## Required Secrets

- AWS credentials (via OIDC or access keys)
- `SENTRY_AUTH_TOKEN` (if using Sentry)
- `SLACK_WEBHOOK_URL` (if using Slack)

## Docker BuildKit Caching

Uses BuildKit with registry caching:
- Cache stored in ECR as `{repository}:cache`
- Speeds up subsequent builds
- Automatic cache management

## Requirements

- Dockerfile in repository root
- App Runner service configured
- ECR repository created
- AWS credentials with ECR and App Runner permissions

## License

MIT

## Author

TahaDekmak
