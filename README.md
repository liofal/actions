# Reusable GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows designed to streamline CI/CD processes, including semantic versioning, Docker image building/publishing, and Helm chart publishing.

## Versioning Convention

These workflows follow standard conventions:
*   **Git Tags & GitHub Releases:** Use a `v` prefix (e.g., `v1.2.3`), managed by `semantic-release`.
*   **Registry Artifacts (Docker Images, Helm Charts):** Use plain semantic versions without prefixes (e.g., `1.2.3`) for compatibility with package managers and tools.

## Available Workflows

1.  **`semantic_release.yml`**: Handles automated versioning and release creation based on conventional commits. Outputs the Git tag version including the `v` prefix.
2.  **`build_and_deploy.yml`**: Builds Docker images using Kaniko and pushes them to container registries (GHCR, Docker Hub). It uses the `v`-prefixed tag for checkout but generates non-prefixed image tags.
3.  **`publish_helm.yml`**: Packages and publishes Helm charts to GHCR as OCI artifacts. It uses the `v`-prefixed tag for checkout but publishes the chart using the non-prefixed version defined in `Chart.yaml`.

---

### 1. Universal Semantic Release (`semantic_release.yml`)

Automates version management and GitHub Release creation using `semantic-release`.

**Usage:**

```yaml
jobs:
  release:
    name: Create Release
    uses: liofal/actions/.github/workflows/semantic_release.yml@main # Or specific tag/commit
    with:
      # Optional: Specify Node.js version
      # node-version: 'lts/*'

      # IMPORTANT: List semantic-release plugins needed by YOUR project.
      # Only include 'semantic-release-helm' if your project has a Helm chart
      # whose Chart.yaml version needs to be updated by semantic-release.
      semantic-release-plugins: >-
        @semantic-release/commit-analyzer
        @semantic-release/release-notes-generator
        # semantic-release-helm # Include only if Helm chart exists in calling repo
        @semantic-release/git
        @semantic-release/github
    permissions:
      # Required permissions for semantic-release plugins
      contents: write # To push tags, releases, and potentially update files (like Chart.yaml)
      issues: write # To comment on issues
      pull-requests: write # To comment on PRs
    secrets: inherit # Pass GITHUB_TOKEN
```

**Inputs:**

*   `node-version` (string, optional, default: `lts/*`): Node.js version to use.
*   `semantic-release-plugins` (string, optional, default: see workflow file): Space-separated list of `semantic-release` plugins to install. **Crucially, only include `semantic-release-helm` if your project uses it.**

**Outputs:**

*   `published` (string: '1' or '0'): Indicates if a new release was published.
*   `version` (string): The new version tag including the `v` prefix (e.g., `v1.2.3`) if published, otherwise empty.

---

### 2. Build and Deploy Docker Images (`build_and_deploy.yml`)

Builds a Docker image using Kaniko and pushes it to GHCR and/or Docker Hub. Typically called after `semantic_release.yml`.

**Usage:**

```yaml
jobs:
  # ... (release job first) ...

  build_deploy:
    name: Build and Deploy Docker Image
    needs: release
    if: needs.release.outputs.published == '1' # Only run if release occurred
    uses: liofal/actions/.github/workflows/build_and_deploy.yml@main # Or specific tag/commit
    permissions:
      contents: read   # Needed for checkout
      packages: write  # Needed to push images to GHCR/Docker Hub
    with:
      # Pass the version tag (e.g., v1.2.3) from the release job
      tag: ${{ needs.release.outputs.version }}
    secrets: inherit # Pass GITHUB_TOKEN and DOCKER_USERNAME, DOCKER_TOKEN if using Docker Hub
```

**Inputs:**

*   `tag` (string, optional): The Git tag (e.g., `v1.2.3`) to check out. The workflow uses this to derive non-prefixed image tags (e.g., `1.2.3`). If provided, images will be pushed.

**Secrets:**

*   Requires `secrets.GITHUB_TOKEN` (passed via `inherit`) for GHCR push.
*   Requires `secrets.DOCKER_USERNAME` and `secrets.DOCKER_TOKEN` (passed via `inherit`) if the `deploy-dockerio` job is used (it's included by default in the reusable workflow).

---

### 3. Publish Helm Chart (`publish_helm.yml`)

Packages a Helm chart and publishes it as an OCI artifact to GHCR. Typically called after `semantic_release.yml` for projects containing Helm charts.

**Usage:**

```yaml
jobs:
  # ... (release job first) ...
  # ... (build_deploy job potentially) ...

  publish_helm_chart:
    name: Publish Helm Chart
    needs: release
    # Condition: Only run if release happened AND Chart.yaml exists in this repo
    if: needs.release.outputs.published == '1' && hashFiles('kube/charts/Chart.yaml') != '' # Adjust path if needed
    uses: liofal/actions/.github/workflows/publish_helm.yml@main # Or specific tag/commit
    permissions:
      contents: read   # Needed for checkout
      packages: write  # Needed to push package to GHCR
    with:
      # Pass the version tag (e.g., v1.2.3) from the release job
      tag: ${{ needs.release.outputs.version }}
      # Optional: Override default chart path if needed
      # chart_path: 'path/to/your/chart'
    secrets: inherit # Pass GITHUB_TOKEN
```

**Inputs:**

*   `tag` (string, required): The Git tag (e.g., `v1.2.3`) to check out. The Helm chart artifact itself will be tagged using the version from `Chart.yaml` (e.g., `1.2.3`).
*   `chart_path` (string, optional, default: `kube/charts`): Path to the Helm chart directory relative to the repository root.

**Secrets:**

*   Requires `secrets.GITHUB_TOKEN` (passed via `inherit`) for GHCR push.

---

## Full Examples

See the following files for complete examples of calling workflows:

*   [`build_and_deploy-example.yml`](./build_and_deploy-example.yml): Example for a project **with** a Helm chart.
*   [`build_and_deploy-no-helm-example.yml`](./build_and_deploy-no-helm-example.yml): Example for a project **without** a Helm chart.
