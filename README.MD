# Setting up "dependabot" to Azure Repos

This is an example on how to enable WhiteSource Renovate bot in a single Git repository hosted in Azure Repos. The `renovate.json` should be placed in the root of your repo and can be configured further by [following the documentation](https://docs.renovatebot.com/configuration-options/).

## Requirements
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- [Azure Devops extension for Azure CLI](https://github.com/Azure/azure-devops-cli-extension)
- [PAT for Azure DevOps CLI](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page#create-personal-access-tokens-to-authenticate-access)
  - `Agent Pools`: Read
  - `Build`: Read & Execute
  - `Code`: Read
  - `Project and Team`: Read
  - `Release`: Read, write, execute, & manage

## Create the pipeline in Azure DevOps

  ```bash
# Login using your AAD account
az login

# Install Azure DevOps CLI
az extension add --name azure-devops

# Login to Azure DevOps using the extension - providing the PAT token created previously
az devops login

# Prepare your environment
AZURE_DEVOPS_ORGANIZATION='<your ado org>'
AZURE_DEVOPS_ACCOUNT='https://dev.azure.com/<your ado org>/'
AZURE_DEVOPS_PROJECT='<your projects name in ado>'
GIT_REPO='<your azure repo name>'
GIT_BRANCH='<your default branch>'

az devops configure --defaults organization=https://dev.azure.com/<your ado org> project=<your projects name in ado>

# Create variable group for dependabot
az pipelines variable-group create --name dependabot --description "Pipeline variables for dependabot." --authorize true --variables LOG_LEVEL=debug ADO_REPO_NAME=$GIT_REPO

# Generate the Personal Access Token for github.com following the instructions from here: https://docs.renovatebot.com/install-gitlab-app/#generate-a-personal-access-token
# Save it as secret variable 'GITHUB_COM_TOKEN'
GROUP_ID=$(az pipelines variable-group list --query "[?name=='dependabot']".id -o tsv)
az pipelines variable-group variable create --group-id $GROUP_ID --name GITHUB_COM_TOKEN --secret true

# Create subfolder for pipelines
FOLDER_PATH='\pipelines'
az pipelines folder create --path "$FOLDER_PATH"

# Create pipeline for dependabot (WhiteSource Renovate)
# https://docs.renovatebot.com/setup-azure-devops/
# Before creating the pipeline, make sure to complete the following, manual step first from ADO UI
# 1) Project Settings: Code > Repositories
# 2) Select the git repo 
# 3) Users > Project Collection Build Service (Qurate)
# 4) Enable the following permissions: Contribute to pull requests, Create branch, Force push
PIPELINE_NAME_DEPENDABOT='dependabot'
PIPELINE_DESCRIPTION_DEPENDABOT='Pipeline for automated dependency updates (WhiteSource Renovate).'
REPO_YAML_PATH_DEPENDABOT='pipelines/dependabot.yml'
az pipelines create --name "$PIPELINE_NAME_DEPENDABOT" \
    --description "$PIPELINE_DESCRIPTION_DEPENDABOT" \
    --repository "$GIT_REPO" \
    --repository-type tfsgit \
    --branch "$GIT_BRANCH" \
    --yml-path "$REPO_YAML_PATH_DEPENDABOT" \
    --folder-path "$FOLDER_PATH"
```
