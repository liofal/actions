# GitHub Actions Workflows

This repository contains GitHub Actions workflows for building and deploying Docker images using Kaniko.

## Workflows

### Build docker images with kaniko

This workflow is triggered on pushes to the `main` branch and on new tags. It builds and deploys Docker images to both GitHub Container Registry (GHCR) and Docker Hub.

#### Workflow Configuration

```yaml
# ...existing code...
name: Build docker images with kaniko

on:
  push:
    branches: [ main ]
    tags:
      - '*'

jobs:
  deploy-ghcr:
    uses: /liofal/action/workflows/reusable_build_and_deploy.yml
    with:
      registry: ghcr.io
      username: ${{ github.actor }}
      password: ${{ secrets.GITHUB_TOKEN }}
      images: ghcr.io/${{ github.repository }}

  deploy-dockerio:
    uses: /liofal/action/workflows/reusable_build_and_deploy.yml
    with:
      registry: docker.io
      username: ${{ secrets.DOCKER_USERNAME }}
      password: ${{ secrets.DOCKER_TOKEN }}
      images: docker.io/${{ github.repository }}
# ...existing code...
```

#### Jobs

- **deploy-ghcr**: Deploys the Docker image to GitHub Container Registry.
- **deploy-dockerio**: Deploys the Docker image to Docker Hub.

#### Secrets

- `GITHUB_TOKEN`: Automatically provided by GitHub for authentication.
- `DOCKER_USERNAME`: Docker Hub username stored in repository secrets.
- `DOCKER_TOKEN`: Docker Hub access token stored in repository secrets.

## Reusable Workflow

The reusable workflow `reusable_build_and_deploy.yml` is used to perform the actual build and deployment steps. It is located in the `liofal/action/workflows` directory.

```yaml
# ...existing code...
```
