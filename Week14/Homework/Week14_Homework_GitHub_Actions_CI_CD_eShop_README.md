# 🚀 Week 14 Homework: CI/CD with GitHub Actions for eShopOnContainers

## 🧠 Objective

This week, you will implement a complete **CI/CD pipeline using GitHub Actions** for the eShopOnContainers application. You'll automate building, testing, securing, and deploying your microservices using the Helm charts created in Week 13, with proper environment promotion and quality gates.

---



## ✅ Tasks

### Step 1: Setup GitHub Repository Structure

Organize your repository for CI/CD:

```bash
# Repository structure
eshop/
├── .github/
│   └── workflows/
│       ├── composite
│             ├── build-push
│                   ├── action.yml
│       ├── basket-api.yml
│       ├── catalog-api.yml
├── src/

```

### Create GitHub Secrets:

Go to your repository → Settings → Secrets and variables → Actions, and add:

```bash
# Container Registry
DOCKER_HUB_USERNAME=your-dockerhub-username
DOCKER_HUB_TOKEN=your-dockerhub-token
```

---

### Step 2: Create Continuous Integration Workflow

#### Build and Test Workflow:
```yaml
# .github/workflows/composite/build-push/action.yml
name: "Build and push image"
description: "Builds and pushes an image to a registry"

inputs:
  service:
    description: "Service to build"
    required: true
  registry_host:
    description: "Image registry host e.g. myacr.azureacr.io"
    required: true
  registry_endpoint:
    description: "Image registry repo e.g. myacr.azureacr.io/eshop"
    required: true
  image_name:
    description: "Name of image"
    required: true
  registry_username:
    description: "Registry username"
    required: true
  registry_password:
    description: "Registry password"
    required: true
  
runs:
  using: "composite"
  steps:
  - name: Enable experimental features for the Docker daemon and CLI
    shell: bash
    run: |
        echo $'{\n  "experimental": true\n}' | sudo tee /etc/docker/daemon.json
        mkdir -p ~/.docker
        echo $'{\n  "experimental": "enabled"\n}' | sudo tee ~/.docker/config.json
        sudo service docker restart
        docker version -f '{{.Client.Experimental}}'
        docker version -f '{{.Server.Experimental}}'

  - name: Login to Container Registry
    uses: docker/login-action@v1
    with:
      registry: ${{ inputs.registry_host }}
      username: ${{ inputs.registry_username }}
      password: ${{ inputs.registry_password }}

  - name: Set branch name as env variable
    run: |
      currentbranch=$(echo ${GITHUB_REF##*/})
      echo "running on $currentbranch"
      echo "BRANCH=$currentbranch" >> $GITHUB_ENV
    shell: bash

  - name: Compose build ${{ inputs.service }}
    shell: bash
    run: sudo -E docker-compose build ${{ inputs.service }}
    working-directory: ./src
    env:
      TAG: ${{ env.BRANCH }}
      REGISTRY: ${{ inputs.registry_endpoint }}

  - name: Compose push ${{ inputs.service }}
    shell: bash
    run: sudo -E docker-compose push ${{ inputs.service }}
    working-directory: ./src
    env:
      TAG: ${{ env.BRANCH }}
      REGISTRY: ${{ inputs.registry_endpoint }}

  - name: Create multiarch manifest
    shell: bash
    run: |
      docker --config ~/.docker manifest create ${{ inputs.registry_endpoint  }}/${{ inputs.image_name }}:${{ env.BRANCH  }} ${{ inputs.registry_endpoint  }}/${{ inputs.image_name  }}:linux-${{ env.BRANCH  }}
      docker --config ~/.docker manifest push ${{ inputs.registry_endpoint  }}/${{ inputs.image_name }}:${{ env.BRANCH  }}
```

---

### Step 3: Create Deployment Workflows for basket-api

#### Development Environment Deployment:
```yaml
# .github/workflows/basket-api.yml
name: basket-api

on:
  workflow_dispatch:
  push:
    branches:
    - main

    paths:
    - src/BuildingBlocks/**
    - src/Services/Basket/**
    - .github/workflows/basket-api.yml
  
  pull_request:
    branches:
    - main

    paths:
    - src/BuildingBlocks/**
    - src/Services/Basket/**
    - .github/workflows/basket-api.yml
env:
  SERVICE: basket-api
  IMAGE: basket.api

jobs:
  BuildLinux:
    runs-on: ubuntu-latest
    if: ${{ github.event_name != 'pull_request' }}
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    - uses: ./.github/workflows/composite/build-push
      with:
        service: ${{ env.SERVICE }}
        registry_host: ${{ secrets.REGISTRY_HOST }}
        registry_endpoint: ${{ secrets.REGISTRY_ENDPOINT }}
        image_name: ${{ env.IMAGE }}
        registry_username: ${{ secrets.USERNAME }}
        registry_password: ${{ secrets.PASSWORD }}
```


---
### Step 4: Create Deployment Workflows for all microservice
Repeat step#3 for all microservices apps


## 📝 Deliverables

Submit all your workflow files


### Screenshots:
- GitHub Actions workflow runs showing successful builds


---


## ⏱️ Deadline

Submit all deliverables before next class. Allow extra time for testing all environments!


