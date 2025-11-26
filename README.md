# Sample Application

This is a demo application that demonstrates using Azure DevOps pipeline templates from a cross-organization repository.

## Overview

This repository contains a simple .NET 8 web application that uses reusable Azure DevOps pipeline templates hosted in a separate GitHub organization (`samcogan-templates`).

## Application Details

- **Framework**: .NET 8
- **Type**: ASP.NET Core Web API
- **Endpoints**:
  - `/` - Returns a hello message
  - `/health` - Returns health status

## Pipeline Structure

The `azure-pipelines.yml` file demonstrates cross-organization template usage:

### Template Repository Reference

```yaml
resources:
  repositories:
    - repository: templates
      type: github
      name: samcogan-templates/azure-pipeline-templates
      endpoint: GitHubConnection
      ref: refs/heads/main
```

### Stages

1. **Build Stage**
   - Code Build (using `code-build.yml` template)
   - Container Build (using `container-build.yml` template)

2. **Security Scan Stage**
   - Security scanning (using `security-scan.yml` template)

3. **Deploy to Development**
   - Container deployment to dev environment (using `deploy-stage.yml` template)

4. **Deploy to Production**
   - Container deployment to production with approval (using `deploy-stage.yml` template)

## Setup Requirements

### 1. GitHub Service Connection

You need to configure a GitHub service connection in Azure DevOps:

1. Go to Project Settings → Service connections
2. Create a new GitHub connection
3. Name it `GitHubConnection` (or update the pipeline to match your connection name)
4. Ensure the connection has access to both organizations:
   - `samcogan-demo` (this repository)
   - `samcogan-templates` (template repository)

### 2. GitHub App Configuration (Recommended for Enterprise)

For cross-organization access in GitHub Enterprise Cloud:

1. **Create a GitHub App** with permissions:
   - Repository Contents: Read
   - Repository Metadata: Read

2. **Install the app in both organizations**:
   - `samcogan-templates`
   - `samcogan-demo`

3. **Configure Azure DevOps** to use the GitHub App for authentication

### 3. Azure Service Connection

Create an Azure service connection:

1. Go to Project Settings → Service connections
2. Create a new Azure Resource Manager connection
3. Name it `AzureDevConnection` (or update the pipeline)
4. Grant access to the Azure resources you'll deploy to

### 4. Azure Container Registry

Set up a service connection for your Azure Container Registry:

1. Create a service connection for ACR
2. Name it `myACR` (or update the pipeline variable)

### 5. Azure Resources

Create the following Azure resources:

- Azure Container Registry (for storing images)
- Azure App Service (for development): `sampleapp-dev`
- Azure App Service (for production): `sampleapp-prod`

Or update the pipeline with your own resource names.

## Cross-Organization Permissions

### Required Permissions

1. **Template Repository** (`samcogan-templates/azure-pipeline-templates`)
   - Must be accessible to the GitHub service connection
   - GitHub App needs read access to Contents and Metadata

2. **This Repository** (`samcogan-demo/sample-app`)
   - Pipeline must have permission to checkout external repositories
   - Service connection must be authorized

### Troubleshooting Access Issues

If you encounter "Repository not found" or permission errors:

1. Verify the GitHub App is installed in both organizations
2. Check the service connection has the correct permissions
3. Ensure the template repository name is correct: `samcogan-templates/azure-pipeline-templates`
4. Verify the branch reference is correct: `refs/heads/main`
5. Check Azure DevOps pipeline permissions allow external repository access

## Running the Application Locally

```bash
# Restore dependencies
dotnet restore

# Run the application
dotnet run

# Build Docker image
docker build -t sampleapp .

# Run container
docker run -p 8080:80 sampleapp
```

## Pipeline Variables to Customize

Update these variables in `azure-pipelines.yml`:

- `imageName`: Your container image name
- `containerRegistry`: Your ACR service connection name
- `GitHubConnection`: Your GitHub service connection name (in resources section)
- `AzureDevConnection`: Your Azure service connection name
- `resourceName`: Your Azure App Service names (dev and prod)

## Benefits of Cross-Org Templates

1. **Centralized Maintenance**: Update templates in one place
2. **Consistency**: Ensure all projects follow the same patterns
3. **Reusability**: Share pipeline logic across organizations
4. **Security**: Centrally managed security scanning and compliance
5. **Governance**: Enforce organizational standards

## Related Resources

- Template Repository: [samcogan-templates/azure-pipeline-templates](https://github.com/samcogan-templates/azure-pipeline-templates)
- Azure DevOps Documentation: [Template usage](https://docs.microsoft.com/azure/devops/pipelines/process/templates)

## License

MIT
