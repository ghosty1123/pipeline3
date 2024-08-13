# CI/CD Pipeline

## Overview

This repository contains a GitHub Actions CI/CD pipeline for a Node.js application. The pipeline is designed to automate the build, test, and deployment processes, as well as provide notifications for failed builds or deployments.

## Pipeline Stages

The CI/CD pipeline consists of the following stages:

1. **Build**: Compiles the source code and generates build artifacts.
2. **Test**: Runs unit tests and performs code quality checks.
3. **Deploy**: Deploys the application to a staging environment.
4. **Notification**: Sends notifications if any stage fails.

## Workflow File

The pipeline is defined in the `.github/workflows/ci-cd-pipeline.yml` file. Here’s a summary of its contents:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js environment
        uses: actions/setup-node@v4
        with:
          node-version: '14'
          cache: 'npm'
          cache-dependency-path: 'package-lock.json'

      - name: Install dependencies
        run: npm install

      - name: Build application
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: ./build

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js environment
        uses: actions/setup-node@v4
        with:
          node-version: '14'
          cache: 'npm'
          cache-dependency-path: 'package-lock.json'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Code quality check
        run: npm run lint

  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to staging
        run: echo "Deploying application to staging environment"

  notification:
    name: Notification
    runs-on: ubuntu-latest
    needs: [build, test, deploy]
    if: failure()
    steps:
      - name: Notify on failure
        uses: actions/github-script@v6
        with:
          script: |
            const message = `Build or deployment failed in workflow run [#${process.env.GITHUB_RUN_NUMBER}](https://github.com/${process.env.GITHUB_REPOSITORY}/actions/runs/${process.env.GITHUB_RUN_ID})`;
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: 1, // Replace with your issue or PR number
              body: message
            });
