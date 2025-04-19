# Build and Push Docker Image Action

This GitHub Action automates the process of building and pushing Docker images to Docker Hub and/or GitHub Container Registry (GHCR). It supports multi-platform builds, generates Open Containers Initiative (OCI) compliant metadata, and optimizes build performance with dynamic caching. Ideal for CI/CD pipelines requiring reliable Docker image publishing.

## Features

- **Multi-Platform Support**: Builds images for multiple architectures (e.g., linux/amd64, linux/arm64) using Docker Buildx and QEMU.
- **Flexible Registry Publishing**: Pushes images to Docker Hub, GHCR, or both, with conditional logic to verify repository existence.
- **Dynamic Tagging**: Generates tags based on branch, tag, semantic version, and commit SHA using docker/metadata-action.
- **OCI Metadata**: Applies labels and annotations like org.opencontainers.image.created, licenses, and version for compliance.
- **Optimized Caching**: Uses dynamic registry-based caching to speed up builds, with cache disabled for tag events to ensure fresh publishes.
- **Environment Variables**: Supports optional .env file loading for configuration.
- **Debugging and Validation**: Includes steps to verify tags, digests, and metadata post-push.

## Prerequisites

- **Docker Hub Credentials**:
  - Create a Docker Hub Personal Access Token (PAT).
  - Add DOCKER_USERNAME and DOCKER_PAT as repository secrets in GitHub.

- **GitHub Container Registry (GHCR)**:
  - Ensure the repository exists and is accessible.
  - Use GITHUB_TOKEN (provided by default) or a custom token with packages:write permission.

- **Permissions**:

```yaml
permissions:
  contents: read
  packages: write
```

**Dockerfile**: A valid Dockerfile in the repository root, supporting multi-stage builds if needed.


## Inputs

| Name               | Description                                                                 | Required | Default                     |
|--------------------|-----------------------------------------------------------------------------|----------|-----------------------------|
| `docker-username`  | Docker Hub username for authentication.                                     | Yes      | -                           |
| `docker-pat`       | Docker Hub Personal Access Token for authentication.                        | Yes      | -                           |
| `image-name`       | Name of the Docker image (e.g., `my-app`).                                  | Yes      | -                           |
| `github-token`     | GitHub token for GHCR authentication.                                       | Yes      | `${{ github.token }}`       |
| `log-level`        | Log level for build process (`debug`, `info`, `warn`, `error`).             | No       | `debug`                     |
| `timezone`         | Timezone for image metadata (e.g., `UTC`, `America/New_York`).              | No       | `UTC`                       |
| `authors`          | Image authors (e.g., `John Doe <john@example.com>`).                        | No       | -                           |
| `licenses`         | Image license (e.g., `MIT`, `Apache-2.0`).                                  | No       | `MIT`                       |
| `vendor`           | Image vendor (e.g., `My Company`).                                          | No       | -                           |
| `maintainers`      | Image maintainers (e.g., `Jane Doe <jane@example.com>`).                    | No       | -                           |
| `description`      | Image description (e.g., `Lightweight linter environment`).                 | No       | -                           |
| `publish-dockerhub`| Whether to publish to Docker Hub (`true` or `false`).                       | No       | `true`                      |
| `publish-ghcr`     | Whether to publish to GitHub Container Registry (`true` or `false`).        | No       | `false`                     |
| `env-file`         | Path to optional `.env` file for environment variables.                     | No       | `.env`                      |
| `title`            | Image title (e.g., `Docker Bullseye Linter`).                               | No       | -                           |
| `version`          | Image version (e.g., `1.0.0`).                                              | No       | -                           |


## Outputs

| Name    | Description                                            |
|---------|--------------------------------------------------------|
| `tags`  | Comma-separated list of generated tags.                |
| `labels`| Generated OCI labels for the image.                    |

## Usage
### Example Workflow

Create a file at `.github/workflows/build.yml`:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0' # Weekly on Sunday
  workflow_dispatch:

permissions:
  contents: read
  packages: write

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Build and Push Docker Image
        uses: your-username/your-repo@main
        with:
          docker-username: ${{ secrets.DOCKER_USERNAME }}
          docker-pat: ${{ secrets.DOCKER_PAT }}
          image-name: my-app
          github-token: ${{ secrets.GITHUB_TOKEN }}
          licenses: MIT
          description: Lightweight environment for my app
          publish-dockerhub: true
          publish-ghcr: true
```

## Example .env File

If using an optional .env file for configuration, create it at the repository root:

```plain
LICENSES=MIT
DESCRIPTION=Lightweight environment for my app
AUTHORS=John Doe <john@example.com>
```

## Multi-Platform Build

The action automatically builds for linux/amd64 and linux/arm64. Ensure your Dockerfile is compatible with these platforms.

## Verifying Tags and Metadata

The action includes debugging steps that output tags, digests, and labels to the GitHub Actions logs. To verify the published image:

```bash
docker pull docker.io/your-username/my-app:latest
docker inspect docker.io/your-username/my-app:latest | grep -E 'org.opencontainers.image'
docker inspect docker.io/your-username/my-app:latest | jq '.[0].Config.Labels'
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository.
2. Create a feature branch (git checkout -b feature/my-feature).
3. Commit your changes (git commit -m 'Add my feature').
4. Push to the branch (git push origin feature/my-feature).
5. Open a Pull Request.

See CONTRIBUTING.md for more details.

## License

This action is licensed under the MIT License.

## Support

For issues or questions, open an issue in the repository or contact the maintainers.
