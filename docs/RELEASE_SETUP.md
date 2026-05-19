# Release Setup Documentation

This document explains how to set up automated releases for the Autotask MCP Server, including GitHub releases, Azure Container Registry publishing, and Azure Container Apps deployment.

## Overview

The release process is automated using GitHub Actions and follows these patterns inspired by the [autotask-node repository](https://github.com/wyre-technology/autotask-node):

1. **Semantic Versioning**: Uses conventional commits and semantic-release
2. **Multi-Platform Testing**: Tests on Node.js 18, 20, and 22
3. **GitHub Releases**: Automated release notes and asset publishing
4. **Container Publishing**: Linux amd64 builds to Azure Container Registry
5. **Security Scanning**: Automated vulnerability scanning with Trivy

## Prerequisites

### Required GitHub Secrets

Set these secrets in your GitHub repository settings (`Settings > Secrets and variables > Actions`):

| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `GITHUB_TOKEN` | Automatically provided by GitHub | *(automatic)* |
| `AZURE_CLIENT_ID` | Azure AD app registration client ID for OIDC login | `11111111-2222-3333-4444-555555555555` |
| `AZURE_TENANT_ID` | Azure tenant ID | `11111111-2222-3333-4444-555555555555` |
| `AZURE_SUBSCRIPTION_ID` | Azure subscription ID | `11111111-2222-3333-4444-555555555555` |
| `ACR_NAME` | Azure Container Registry name | `ion247` |
| `AZURE_CONTAINER_APP_NAME` | Azure Container App name to update during deploy | `autotask-prod` |
| `AZURE_CONTAINER_APP_RESOURCE_GROUP` | Resource group containing the Azure Container App | `production-rg` |

### Azure Deployment Setup

1. Create or reuse an Azure AD application for GitHub OIDC.
2. Grant it access to the subscription and resource group used by the workflow.
3. Add `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, and `AZURE_SUBSCRIPTION_ID` as repository secrets.
4. Add the Azure Container Registry name as `ACR_NAME`.
5. Add `AZURE_CONTAINER_APP_NAME` and `AZURE_CONTAINER_APP_RESOURCE_GROUP` so the deploy step knows which app to update.

### Azure Container Registry Access

The workflow does not need a static registry password. It signs in to Azure via OIDC, requests a short-lived ACR access token with `az acr login --expose-token`, and uses that token for `docker/login-action`.

## Workflow Files

### `.github/workflows/release.yml`

Main release workflow that:
- Runs on pushes to `main` branch
- Tests across multiple Node.js versions
- Creates GitHub releases using semantic-release
- Builds and publishes container images to Azure Container Registry
- Deploys Azure Container Apps
- Runs security scans

### `.github/workflows/test.yml`

Pull request testing workflow that:
- Runs on pull requests
- Tests across multiple Node.js versions
- Runs code quality checks
- Tests Docker builds

## Semantic Release Configuration

### `.releaserc.json`

Configures semantic-release with:
- Branch configuration for main, next, beta, alpha
- Plugins for changelog generation and GitHub releases
- Automated release notes based on commit messages

### Commit Message Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```bash
# Patch release (1.0.0 → 1.0.1)
fix: resolve API timeout issues

# Minor release (1.0.0 → 1.1.0) 
feat: add project search functionality

# Major release (1.0.0 → 2.0.0)
feat: restructure API endpoints

BREAKING CHANGE: API endpoints have changed
```

## Docker Configuration

### Multi-Stage Dockerfile

The Dockerfile uses multi-stage builds for:
- **Builder stage**: Compiles TypeScript and installs dependencies
- **Production stage**: Minimal runtime image with only production dependencies
- **Security**: Runs as non-root user with proper file permissions
- **Observability**: Includes health checks and proper logging

### Azure Container Registry Publishing

Images are published to Azure Container Registry with multiple tags:
- `latest`: Latest stable release
- `v1.0.1`: Specific version tag
- Platform support: `linux/amd64`

### Image Labels

Images include comprehensive OCI labels:
- Version, commit SHA, build date
- Source repository and documentation links
- Vendor and license information

## Release Process

### Automated Release (Recommended)

1. **Make Changes**: Develop features/fixes on feature branches
2. **Create PR**: Submit pull request to `main` branch
3. **Review & Test**: GitHub Actions run automatic tests
4. **Merge PR**: Merge to `main` using conventional commit messages
5. **Automatic Release**: GitHub Actions automatically:
   - Determines version bump based on commits
   - Creates GitHub release with notes
   - Builds and pushes container images to ACR
   - Deploys the latest image to Azure Container Apps
   - Runs security scans

### Manual Release Preparation

Use the preparation script:

```bash
# Make script executable
chmod +x scripts/prepare-release.sh

# Run preparation checks
./scripts/prepare-release.sh
```

The script will:
- ✅ Verify all tools are installed
- ✅ Check Node.js version compatibility
- ✅ Run linting and tests
- ✅ Build the project
- ✅ Test Docker build
- ✅ Verify Git status

### Manual Release Commands

If needed, you can manually create releases:

```bash
# Create GitHub release
gh release create v1.0.1 --auto --notes

# Build and push Docker image
docker build -t <acr-name>.azurecr.io/autotask-mcp:v1.0.1 .
docker push <acr-name>.azurecr.io/autotask-mcp:v1.0.1
docker tag <acr-name>.azurecr.io/autotask-mcp:v1.0.1 <acr-name>.azurecr.io/autotask-mcp:latest
docker push <acr-name>.azurecr.io/autotask-mcp:latest
```

## Monitoring and Maintenance

### Release Monitoring

Monitor releases through:
- **GitHub**: Check Actions tab for workflow status
- **Azure Container Registry**: Verify image tags in your ACR instance
- **Azure Container Apps**: Check the active revision and image for the deployed app

### Security Scanning

Automated security scanning with Trivy:
- Scans Docker images for vulnerabilities
- Results uploaded to GitHub Security tab
- Runs after each release

### Failed Release Recovery

If a release fails:

1. **Check Logs**: Review GitHub Actions logs for errors
2. **Fix Issues**: Address any build, test, or deployment issues
3. **Retry Release**: Push a fix with conventional commit message
4. **Manual Intervention**: If needed, manually create release components

Common failure points:
- **Azure Login**: OIDC configuration or secret mismatches
- **ACR Publishing**: Registry access or build issues
- **Security Scan**: Critical vulnerabilities found

## Version Management

### Pre-release Branches

Support for pre-release versions:

```bash
# Beta release
git checkout -b beta
git push origin beta
# Creates version like 1.1.0-beta.1

# Alpha release  
git checkout -b alpha
git push origin alpha
# Creates version like 1.1.0-alpha.1
```

### Version Rollback

If you need to rollback a release:

```bash
# Revert Git tag
git tag -d v1.0.1
git push origin :refs/tags/v1.0.1

# Remove or replace the deployed image tag in ACR according to your registry retention policy
```

## Troubleshooting

### Common Issues

#### Semantic Release Fails
```bash
# Error: No GITHUB_TOKEN
# Solution: Token is automatically provided, check permissions
```

#### Docker Build Fails
```bash
# Error: Architecture not supported
# Solution: Check build-args and platform specification

# Error: Authentication failed
# Solution: Verify Azure OIDC configuration and ACR permissions
```

#### Tests Fail in CI
```bash
# Error: Node version mismatch
# Solution: Update Node.js version in workflow

# Error: Dependencies not found
# Solution: Ensure npm ci runs before build/test
```

### Debug Mode

Enable debug logging in GitHub Actions:

1. Go to repository Settings > Secrets
2. Add secret: `ACTIONS_STEP_DEBUG` = `true`
3. Re-run workflow for detailed logs

## Best Practices

### Commit Messages
- Use conventional commit format
- Write clear, descriptive messages
- Reference issues when applicable

### Branch Management
- Keep `main` branch stable
- Use feature branches for development
- Require PR reviews before merging

### Release Planning
- Group related changes in single release
- Test thoroughly before releasing
- Document breaking changes clearly

### Security
- Regularly update dependencies
- Monitor security scan results
- Rotate access tokens periodically

## Related Documentation

- [Docker Usage Guide](DOCKER_USAGE.md)
- [Main README](README.md)
- [Autotask Node Library](https://github.com/wyre-technology/autotask-node)
- [Semantic Release Documentation](https://semantic-release.gitbook.io/)
- [Conventional Commits](https://www.conventionalcommits.org/)

For questions or issues with the release process, please open an issue in the GitHub repository. 