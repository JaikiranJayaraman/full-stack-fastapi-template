# CI/CD Pipeline Documentation

## Overview
This document describes the CI/CD pipeline setup for the full-stack-fastapi-template project.

## Pipeline Architecture
```
Push/PR → Tests → SonarQube → Deploy
                ↓
        Dependency Scan
                ↓
        Container Scan
                ↓
        Slack Notification (on failure)
```

## Workflows

### 1. test-backend.yml
- Triggers on push to master and pull requests
- Sets up Python 3.10 and uv
- Starts DB and mailcatcher via Docker
- Runs migrations
- Runs tests with coverage
- Fails if coverage is below 90%

### 2. deploy-staging.yml
- Triggers on push to master
- Requires test-backend to pass first (needs gate)
- Builds and deploys Docker images to staging server

### 3. sonarqube.yml
- Triggers on push to master and pull requests
- Runs after tests pass
- Scans Python and TypeScript code
- Enforces quality gate

### 4. dependency-scan.yml
- Triggers on push to master and pull requests
- Scans Python dependencies using pip-audit
- Scans frontend dependencies using npm audit
- Reports HIGH and CRITICAL vulnerabilities

### 5. container-scan.yml
- Triggers on push to master and pull requests
- Builds backend and frontend Docker images
- Scans images using Trivy
- Reports HIGH and CRITICAL vulnerabilities

### 6. notify.yml
- Triggers when any pipeline fails
- Sends Slack notification with failure details
- Includes repo, branch, and workflow link

## GitHub Secrets Required
| Secret | Description |
|--------|-------------|
| SONAR_TOKEN | SonarQube authentication token |
| SONAR_HOST_URL | SonarQube server URL |
| SLACK_WEBHOOK_URL | Slack incoming webhook URL |
| DOMAIN_STAGING | Staging domain name |
| STACK_NAME_STAGING | Docker stack name for staging |
| SECRET_KEY | Django secret key |
| FIRST_SUPERUSER | Admin email |
| FIRST_SUPERUSER_PASSWORD | Admin password |
| POSTGRES_PASSWORD | Database password |

## SonarQube Setup
1. SonarQube runs on http://44.210.118.14:9000
2. Project key: full-stack-fastapi
3. Quality gate enforced on every PR

## Local Development
```bash
# Start all services
docker compose up -d

# Run tests
cd backend && uv run bash scripts/tests-start.sh

# Run dependency scan
pip install pip-audit && pip-audit

# Run container scan
docker build -t backend-image ./backend
trivy image backend-image
```
