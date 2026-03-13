# CI/CD Pipeline Documentation

## Overview
This document describes the CI/CD pipeline setup for the full-stack-fastapi-template project (FastAPI backend + React frontend).

## Pipeline Architecture
```
Push/PR → Tests → SonarQube → Deploy
        → Dependency Scan
        → Container Scan
        → Notify on Failure (GitHub Issue)
```

## Workflows

### 1. test-backend.yml (existing)
- Triggers on push to master and pull requests
- Sets up Python 3.10 and uv
- Starts DB and mailcatcher via Docker
- Runs migrations and tests with coverage
- Fails if coverage is below 90%

### 2. deploy-staging.yml (modified)
- Triggers on push to master
- Requires test-backend to pass first (needs gate added)
- Builds and deploys Docker images to staging server
- Requires self-hosted runner with staging label

### 3. pre-commit.yml (existing)
- Triggers on pull requests
- Runs Ruff (Python linting)
- Runs Prettier/ESLint (frontend formatting)
- Auto-commits formatting fixes

### 4. sonarqube.yml (new)
- Triggers on push to master and pull requests
- Scans Python and TypeScript code
- Reports Security, Reliability, Maintainability
- Quality Gate enforced on every push
- SonarQube running on EC2: http://44.210.118.14:9000

### 5. dependency-scan.yml (new)
- Triggers on push to master and pull requests
- Scans Python dependencies using pip-audit
- Scans frontend dependencies using npm audit
- Reports HIGH and CRITICAL vulnerabilities

### 6. container-scan.yml (new)
- Triggers on push to master and pull requests
- Builds backend and frontend Docker images
- Scans images using Trivy
- Reports HIGH and CRITICAL vulnerabilities

### 7. notify.yml (new)
- Triggers when Test Backend, Dependency Scan, or Container Scan fails
- Creates a GitHub Issue with failure details
- Includes workflow name, branch, and link to failed run

## GitHub Secrets Required
| Secret | Description |
|--------|-------------|
| SONAR_TOKEN | SonarQube authentication token |
| SONAR_HOST_URL | SonarQube server URL (http://44.210.118.14:9000) |
| DOMAIN_STAGING | Staging domain name |
| STACK_NAME_STAGING | Docker stack name for staging |
| SECRET_KEY | FastAPI secret key |
| FIRST_SUPERUSER | Admin email |
| FIRST_SUPERUSER_PASSWORD | Admin password |
| POSTGRES_PASSWORD | Database password |

## SonarQube Setup
1. SonarQube Community Edition running via Docker on EC2
2. URL: http://44.210.118.14:9000
3. Project key: full-stack-fastapi
4. Analyzes: backend/app + frontend/src
5. Quality Gate: Passed ✅

## Local Development
```bash
# Start all services
docker compose up -d

# Run tests
cd backend && uv run bash scripts/tests-start.sh

# Run dependency scan
pip install pip-audit && pip-audit

# Run container scan
docker build -t backend-image -f backend/Dockerfile .
trivy image backend-image
```
