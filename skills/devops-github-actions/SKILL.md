---
name: devops-github-actions
description: GitHub Actions expert. Covers workflows, jobs, steps, actions, secrets, matrix builds, and CI/CD pipelines. Use for any GitHub Actions question.
model: opus
---

# GitHub Actions Skill

Apply these GitHub Actions patterns and practices when writing CI/CD workflows.

## Code Style

- Use meaningful workflow and job names
- Use consistent naming: kebab-case for files, snake_case for jobs
- Pin action versions to full SHA or major version
- Keep workflows focused on single purpose
- Use reusable workflows for shared logic

## Workflow Structure

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
```

## Monorepo Patterns

### Path Filtering
```yaml
on:
  push:
    paths:
      - 'frontend/**'
      - 'package.json'
      - 'pnpm-lock.yaml'

  pull_request:
    paths:
      - 'frontend/**'
```

### Matrix Strategy
```yaml
jobs:
  test-services:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        service: [auth, users, orders]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.24'
      - name: Test ${{ matrix.service }}
        run: |
          cd services/${{ matrix.service }}
          make test
```

### Conditional Jobs
```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      services: ${{ steps.filter.outputs.services }}
    steps:
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            frontend:
              - 'frontend/**'
            services:
              - 'services/**'

  build-frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm build:web

  build-services:
    needs: changes
    if: needs.changes.outputs.services == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make build-services
```

## Caching

### Node/pnpm
```yaml
- uses: pnpm/action-setup@v4
  with:
    version: 9
- uses: actions/setup-node@v4
  with:
    node-version: '22'
    cache: 'pnpm'
```

### Go
```yaml
- uses: actions/setup-go@v5
  with:
    go-version: '1.24'
    cache: true
    cache-dependency-path: services/**/go.sum
```

### Custom Cache
```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.cache/golangci-lint
    key: golangci-lint-${{ hashFiles('services/**/go.mod') }}
    restore-keys: |
      golangci-lint-
```

## Secrets and Environment

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1
      - run: aws lambda update-function-code ...

  # Environment variables
  test:
    runs-on: ubuntu-latest
    env:
      CI: true
      NODE_ENV: test
    steps:
      - run: pnpm test
```

## OIDC Authentication (AWS)

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-actions
          aws-region: us-east-1
```

## Reusable Workflows

### Define Reusable Workflow
```yaml
# .github/workflows/build-go-service.yml
name: Build Go Service

on:
  workflow_call:
    inputs:
      service_name:
        required: true
        type: string
      go_version:
        required: false
        type: string
        default: '1.24'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: ${{ inputs.go_version }}
      - name: Build
        run: |
          cd services/${{ inputs.service_name }}
          make build
      - uses: actions/upload-artifact@v4
        with:
          name: ${{ inputs.service_name }}-lambda
          path: services/${{ inputs.service_name }}/lambda.zip
```

### Use Reusable Workflow
```yaml
jobs:
  build-auth:
    uses: ./.github/workflows/build-go-service.yml
    with:
      service_name: auth

  build-users:
    uses: ./.github/workflows/build-go-service.yml
    with:
      service_name: users
```

## Deployment Patterns

### Deploy on Release
```yaml
on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to production
        run: ./scripts/deploy.sh
        env:
          VERSION: ${{ github.event.release.tag_name }}
```

### Environment Protection
```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - run: ./deploy.sh staging

  deploy-prod:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    steps:
      - run: ./deploy.sh production
```

## Composite Actions

```yaml
# .github/actions/setup-project/action.yml
name: Setup Project
description: Setup Node, pnpm, and install dependencies
model: opus

runs:
  using: composite
  steps:
    - uses: pnpm/action-setup@v4
      with:
        version: 9
    - uses: actions/setup-node@v4
      with:
        node-version: '22'
        cache: 'pnpm'
    - run: pnpm install --frozen-lockfile
      shell: bash
```

## Quality Checklist

When writing GitHub Actions workflows, verify:
- [ ] Permissions are minimal (least privilege)
- [ ] Actions pinned to specific versions
- [ ] Secrets not exposed in logs
- [ ] Concurrency configured to prevent duplicate runs
- [ ] Caching configured for dependencies
- [ ] Path filters limit unnecessary runs
- [ ] Fail-fast appropriate for matrix jobs
- [ ] Artifacts uploaded for build outputs
- [ ] Environment protection for production deploys
