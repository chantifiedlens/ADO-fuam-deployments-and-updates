# FUAM Deployment and Updates Pipeline

This repository contains an Azure DevOps YAML pipeline for deploying or updating the **Fabric Unified Admin Monitoring (FUAM)** solution.

## About FUAM

The Fabric Unified Admin Monitoring (FUAM) solution provides comprehensive monitoring capabilities for Microsoft Fabric environments. The official FUAM solution can be found at:

**https://github.com/microsoft/fabric-toolbox/tree/main/monitoring/fabric-unified-admin-monitoring**

## Overview

This repository is specifically designed to work with **YAML Pipelines in Azure DevOps**. It automates the deployment and update process of the FUAM solution to your Microsoft Fabric workspace, handling:

- Creation of required connections (Power BI and Fabric service API connections)
- Workspace creation and management
- Required permissions configuration
- Notebook deployment and execution

## Prerequisites

Before using this pipeline, ensure you have:

1. **Azure DevOps** organization and project access
2. **Microsoft Fabric** capacity available
3. **Service Principal** with appropriate permissions:
   - Fabric Admin or Power Platform Admin permissions
   - Azure AD application with Client ID and Client Secret
4. **Entra ID User Object ID** for the user who will be the FUAM admin

## Pipeline Parameters

The pipeline requires the following parameters:

| Parameter Name | Type | Description | Default Value |
|----------------|------|-------------|---------------|
| `new_installation` | boolean | Indicates if this is a new installation. If `false`, read FUAM update notes before proceeding. | `true` |
| `azure_tenant_id` | string | Your Azure Tenant ID (GUID format) | `00000000-0000-0000-0000-000000000000` |
| `azure_client_id` | string | Client ID of the service principal used for FUAM connections and deployment | `00000000-0000-0000-0000-000000000000` |
| `azure_client_secret` | string | Client Secret of the service principal | `"SECRETVALUE"` |
| `EntraObjectId` | string | Object ID of the Entra user to add as FUAM admin | `00000000-0000-0000-0000-000000000000` |
| `WorkspaceName` | string | Name of the Fabric workspace to deploy FUAM to | `"FUAM"` |
| `CapacityName` | string | Name of the Fabric capacity the workspace will use | `"CapacityName"` |

## Variable Groups

The pipeline uses Azure DevOps Variable Groups to store configuration values. You must create a variable group named `fuam-ns` (or modify the pipeline to reference your own variable group name).

### Required Variables

Add the following variables to your variable group:

| Variable Name | Description | Recommended Value |
|---------------|-------------|-------------------|
| `pbi_connection_name` | Name of the Power BI connection to create | `fuam pbi-service-api admin` |
| `pbibaseUrl` | Base URL for Power BI connection | `https://api.powerbi.com/v1.0/myorg/admin` |
| `pbiaudience` | Audience for Power BI connection | `https://analysis.windows.net/powerbi/api` |
| `fabric_connection_name` | Name of the Fabric connection to create | `fuam fabric-service-api admin` |
| `fabricbaseUrl` | Base URL for Fabric connection | `https://api.fabric.microsoft.com/v1/admin` |
| `fabricaudience` | Audience for Fabric connection | `https://api.fabric.microsoft.com` |
| `DeployURL` | URL to download the Deploy_FUAM notebook from the latest release | `https://raw.githubusercontent.com/microsoft/fabric-toolbox/main/monitoring/fabric-unified-admin-monitoring/scripts/Deploy_FUAM.ipynb` |

### Creating a Variable Group

1. In Azure DevOps, navigate to **Pipelines** → **Library**
2. Click **+ Variable group**
3. Name it `fuam-ns`
4. Add each variable listed above with appropriate values
5. Click **Save**

## Adding the Pipeline to Azure DevOps

Follow these steps to add this repository as a YAML pipeline in Azure DevOps:

### Step 1: Connect Your Repository

1. Navigate to your Azure DevOps project
2. Go to **Pipelines** → **Pipelines**
3. Click **New Pipeline** (or **Create Pipeline** if this is your first pipeline)

### Step 2: Select Your Source

Choose where your code is located:
- **Azure Repos Git** - if you've imported this repository into Azure Repos
- **GitHub** - if you're using GitHub
- **Other Git** - for other Git providers

### Step 3: Select Repository

1. Select the repository `ADO-fuam-deployments-and-updates`
2. Authenticate if required

### Step 4: Configure Pipeline

1. Select **Existing Azure Pipelines YAML file**
2. Choose the branch (typically `main` or `master`)
3. Select the path: `/AzureDevOpsTemplates/fuam-deployments-and-updates.yml`
4. Click **Continue**

### Step 5: Review and Run

1. Review the pipeline YAML
2. Click **Run** to execute the pipeline, or click the dropdown and select **Save** to save without running
3. When running, you'll be prompted to provide parameter values:
   - Fill in all required parameters with your environment-specific values
   - Ensure the service principal credentials are correct
   - Verify the Entra Object ID is correct

### Step 6: Configure Pipeline Settings (Optional)

After creating the pipeline, you may want to:

1. **Rename the pipeline**: Click on the pipeline → Three dots (⋯) → Rename/move
2. **Add to folder**: Organize your pipelines into folders
3. **Configure triggers**: By default, `trigger: none` means manual execution only

## Pipeline Stages

The pipeline executes in three stages:

### 1. CreateConnections
- Installs required Python libraries and Fabric CLI
- Creates Power BI and Fabric service API connections
- Assigns permissions to the specified Entra user

### 2. WorkspaceManagement
- Creates new workspace (if `new_installation = true`)
- Configures workspace permissions
- Prepares workspace for notebook deployment

### 3. RunNotebook
- Downloads the latest Deploy_FUAM notebook from the release
- Imports deployment and post-deployment notebooks
- Executes the notebooks to complete FUAM installation

## Agent Pool

By default, the pipeline uses Microsoft-hosted agents (`ubuntu-latest`). If you want to use a self-hosted agent pool:

1. Uncomment the self-hosted pool configuration in the YAML file
2. Comment out the Microsoft-hosted configuration
3. Update the pool name to match your self-hosted pool

```yaml
pool: 
  # vmImage: 'ubuntu-latest'
  name: 'YourSelfHostedPoolName'
```

## Troubleshooting

### Common Issues

1. **Authentication Failures**: Verify service principal credentials and permissions
2. **Workspace Creation Failures**: Ensure the capacity name is correct and available
3. **Connection Creation Failures**: Check that variable group values are correct
4. **Notebook Import Failures**: Verify the `DeployURL` variable points to a valid release

### Logs

Each pipeline run provides detailed logs:
- Navigate to the pipeline run
- Click on each stage/job to view detailed execution logs
- Check for errors in PowerShell tasks

## Updates vs. New Installations

- **New Installation** (`new_installation: true`): Creates everything from scratch
- **Update** (`new_installation: false`): 
  - Removes existing Deploy_FUAM notebook
  - Imports latest version
  - Preserves existing workspace and connections

Always review the FUAM update notes before performing an update to understand breaking changes or migration requirements.

## Security Considerations

- Store `azure_client_secret` as a **secret variable** in your variable group
- Restrict access to the variable group to authorized personnel only
- Use service connections for enhanced security where possible
- Regularly rotate service principal credentials

## Support and Contributing

For issues related to the FUAM solution itself, please refer to:
- [FUAM GitHub Repository](https://github.com/microsoft/fabric-toolbox/tree/main/monitoring/fabric-unified-admin-monitoring)

For issues with this pipeline implementation, please open an issue in this repository.

## License

This pipeline implementation follows the licensing of the FUAM solution. Please refer to the official FUAM repository for license details.
