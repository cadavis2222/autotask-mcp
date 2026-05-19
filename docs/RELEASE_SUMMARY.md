# 🚀 Release Setup Complete!

Your Autotask MCP Server is now ready for automated releases! Here's what has been set up:

## ✅ What's Been Created

### GitHub Actions Workflows
- **`.github/workflows/release.yml`** - Main release automation
  - Tests on Node.js 18, 20, 22
  - Creates GitHub releases with semantic-release
  - Builds and pushes Docker images to Azure Container Registry
  - Deploys the latest image to Azure Container Apps
  - Runs security scans with Trivy

- **`.github/workflows/test.yml`** - Pull request testing
  - Comprehensive testing on multiple Node.js versions
  - Code quality checks and coverage
  - Docker build testing

### Release Configuration
- **`.releaserc.json`** - Semantic release configuration
  - Supports main, next, beta, alpha branches
  - Automated changelog generation
  - GitHub release creation
  - GitHub-only release publishing

### Docker Setup
- **Enhanced Dockerfile** - Production-ready containerization
  - Multi-stage builds for optimization
  - Multi-architecture support (amd64/arm64)
  - Security hardening with non-root user
  - Comprehensive OCI labels
  - Health checks

- **`src/wrapper.js`** - Docker container wrapper
  - Proper signal handling for containers
  - Graceful shutdown management
  - Error handling and logging

### Documentation & Scripts
- **`RELEASE_SETUP.md`** - Complete release documentation
- **`DOCKER_USAGE.md`** - Comprehensive Docker usage guide
- **`scripts/prepare-release.sh`** - Release preparation script
- **`RELEASE_SUMMARY.md`** - This summary document

### Package Updates
- Added semantic-release dependencies to `package.json`
- Updated TypeScript types with proper `projectType` field

## 🔧 Required Setup Steps

### 1. GitHub Repository Secrets

Add these secrets in your GitHub repository (`Settings > Secrets and variables > Actions`):

| Secret | Required | Description |
|--------|----------|-------------|
| `GITHUB_TOKEN` | ✅ Auto | Automatically provided |
| `AZURE_CLIENT_ID` | ✅ Required | Azure AD application client ID for OIDC login |
| `AZURE_TENANT_ID` | ✅ Required | Azure tenant ID |
| `AZURE_SUBSCRIPTION_ID` | ✅ Required | Azure subscription ID |
| `ACR_NAME` | ✅ Required | Azure Container Registry name |
| `AZURE_CONTAINER_APP_NAME` | ✅ Required | Azure Container App name to update during deploy |
| `AZURE_CONTAINER_APP_RESOURCE_GROUP` | ✅ Required | Resource group containing the Azure Container App |

### 2. Azure Deployment Setup

1. Create or reuse an Azure AD app registration authorized for GitHub OIDC.
2. Add `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, and `AZURE_SUBSCRIPTION_ID` as repository secrets.
3. Add the Azure Container Registry name as `ACR_NAME`.
4. Add `AZURE_CONTAINER_APP_NAME` and `AZURE_CONTAINER_APP_RESOURCE_GROUP` for the deploy target.

## 🚀 How to Release

### Automated Release (Recommended)

1. **Develop**: Make changes on feature branches
2. **PR**: Create pull request to `main` 
3. **Merge**: Use conventional commit messages:
   ```bash
   feat: add new functionality      # Minor version bump
   fix: resolve bug in API          # Patch version bump
   feat!: breaking API change       # Major version bump
   ```
4. **Automatic**: GitHub Actions handles the rest!

### Manual Release Preparation

```bash
# Run the preparation script
./scripts/prepare-release.sh

# If all checks pass, commit and push
git add .
git commit -m "feat: prepare for release"
git push origin main
```

## 📦 What Gets Published

### GitHub Releases
- ✅ Automated release notes
- ✅ Version tags (e.g., `v1.0.2`)
- ✅ Distribution files as assets

### Azure Container Registry Images
- ✅ `<acr-name>.azurecr.io/autotask-mcp:latest`
- ✅ `<acr-name>.azurecr.io/autotask-mcp:v1.0.2`
- ✅ Linux AMD64 image with metadata labels

### Azure Container Apps Deployment
- ✅ Updates the production container app to the latest pushed image
- ✅ Uses the `latest` tag for the production rollout

## 🔍 Monitoring Releases

### GitHub Actions
- Check the **Actions** tab for workflow status
- Review logs for any failures
- Security scan results in **Security** tab

### Azure Container Registry
- Verify the `latest` and versioned tags in your ACR instance
- Confirm image metadata and push timestamps

### Azure Container Apps
- Check the active revision in the Container App
- Confirm the deployed image matches the latest release

## 🛠 Using Released Images

### Quick Start with Docker
```bash
# Pull the latest image
docker pull <acr-name>.azurecr.io/autotask-mcp:latest

# Run with environment variables
docker run -d \
  --name autotask-mcp \
  -e AUTOTASK_USERNAME="your-user@company.com" \
  -e AUTOTASK_SECRET="your-secret" \
  -e AUTOTASK_INTEGRATION_CODE="your-code" \
  <acr-name>.azurecr.io/autotask-mcp:latest
```

### Docker Compose
```yaml
services:
  autotask-mcp:
    image: <acr-name>.azurecr.io/autotask-mcp:latest
    environment:
      - AUTOTASK_USERNAME=${AUTOTASK_USERNAME}
      - AUTOTASK_SECRET=${AUTOTASK_SECRET}
      - AUTOTASK_INTEGRATION_CODE=${AUTOTASK_INTEGRATION_CODE}
    restart: unless-stopped
```

## 🔒 Security Features

- ✅ **Trivy Scanning**: Automatic vulnerability detection
- ✅ **Multi-stage Builds**: Minimal attack surface
- ✅ **Non-root User**: Container security hardening
- ✅ **Secret Management**: Secure credential handling
- ✅ **Dependency Scanning**: Regular security updates

## 📋 Next Steps

1. **Set up secrets** in your GitHub repository
2. **Test the workflow** by making a small change and pushing to main
3. **Monitor the release** in GitHub Actions
4. **Verify container images** are published to Azure Container Registry
5. **Confirm the Azure Container App** is running the new image

## 🆘 Troubleshooting

If releases fail:

1. **Check GitHub Actions logs** for detailed error messages
2. **Verify secrets** are correctly set in repository settings
3. **Run preparation script** locally to test build/test issues
4. **Review release documentation** for common solutions

For detailed troubleshooting, see:
- [RELEASE_SETUP.md](RELEASE_SETUP.md) - Complete setup guide
- [DOCKER_USAGE.md](DOCKER_USAGE.md) - Docker-specific help

## 🎉 Success Indicators

Your release setup is working when you see:

- ✅ GitHub Actions workflows completing successfully
- ✅ New releases appearing in GitHub Releases tab
- ✅ Container images published to Azure Container Registry
- ✅ Azure Container App revisions updating to the new image
- ✅ Security scans completing without critical issues
- ✅ Proper version tagging following semantic versioning

**Your autotask-mcp server is now ready for production deployment!** 🚀

---

*Based on patterns from [autotask-node](https://github.com/wyre-technology/autotask-node) repository* 