# Enterprise CI/CD Pipeline Templates

[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2.0-2088FF.svg)](https://docs.github.com/en/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A **comprehensive collection of production-grade CI/CD pipelines** designed for enterprise scale. This repository contains reusable workflows, composite actions, and automation scripts for Python, Node.js, and Docker-based applications, incorporating security scanning, linting, and automated releases.

## 🚀 Features

- **Reusable Workflows**: Centralized workflow templates for CI/CD consistency across microservices.
- **Polyglot Support**: Optimized pipelines for Python (Pytest, Black), Node.js (Jest, ESLint), and Go.
- **Security First**: Integrated Trivy container scanning, SonarQube analysis, and dependency auditing.
- **Docker Automation**: Automated Docker image building, tagging, and pushing to registries (GHCR/ECR).
- **Composite Actions**: Custom actions to abstract complex setup steps and reduce boilerplate.
- **Release Management**: Semantic versioning and automated release numbering.

## 📁 Project Structure

```
cicd-github-actions-enterprise-pipeline/
├── .github/
│   ├── workflows/        # Reusable workflow templates
│   │   ├── ci-python.yml
│   │   ├── ci-node.yml
│   │   └── cd-docker.yml
│   └── actions/          # Custom composite actions
│       └── setup-env/
├── scripts/              # Utility scripts for local dev
└── Makefile              # Standardized task runner
```

## 🛠️ Usage

### Using a Reusable Workflow

```yaml
jobs:
  ci:
    uses: Shivay00001/cicd-github-actions-enterprise-pipeline/.github/workflows/ci-python.yml@main
    with:
      python-version: '3.11'
```

## 📄 License

MIT License
