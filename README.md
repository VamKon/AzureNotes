# AZ-400
Gotcha's for AZ-400 prep

## ARM Templates
To deploy complex solutions, you can break a template into many templates, and deploy these templates through a main template.

## Azure Key Vault
Need to enable AzureKeyVault for template deployment so that ARM Template can read secrets from KeyVault (enabledForTemplateDeployment property)

## AKS
* Scaling nodes = az aks scale -g myResourceGroup -n myAKSCluster --node-count 3
* To allow an AKS cluster to interact with other Azure resources, an Azure Active Directory service principal is used.

## Helm
* Helm init - install tiller on AKS
* Helm install - install helm charts on AKS

## Azure Devops
### Artifacts
* Public Feeds - you need to have a public project to create a public feed. You cannot convert an existing project scoped feed to a public feed.
* .npmrc (npm config) file:
  * in user’s home folder will contain credentials for all of the registries needed. ($Home = linux, $env.Home = win)
  * In project root beside project.json will contain registry info

