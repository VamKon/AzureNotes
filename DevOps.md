# DevOps

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
  * ![](/images/Artifacts_FeedPermissionsOverview.JPG)

### CI
* Java - To deploy complex solutions, you can break a template into many templates, and deploy these templates through a main template.
* Checking the “Batch” option in the triggers tab of a build will reduce the number of times the CI builds will run if multiple check ins are being made. when a pipeline is running, the system waits until the run is completed, then starts another run with all changes that have not yet been built.

### CD
* Pre/Post Deployment Approvals - Manual Intervention
* Pre/Post Deployment Gates - Automation

### Jenkins
* To allow Jenkins access to ACR, you need to create a service principal in Azure. In Jenkins you need to create a credential resource for the ACR service principal.
* Install Azure Storage Plugin for Jenkins to store build output in Azure Storage.

### Repository
* User “Choose import from Git repository” to import an existing repo in GitHub to Azure DevOps
* The Azure Boards app for GitHub is the preferred method for integrating Azure Boards with GitHub. By connecting you can support linking between GitHub commits and pull requests to work items in Azure.
* Adding a repo to Azure boards - Add it from GitHub Connections in Azure Devops, project settings.

### Agile
* Cumulative Flow Diagram - Count of work items (over time) for each column of a kanban board
* Cycle Time - Time it takes for your team to complete work items once they begin actively working on them
* Lead Time - Time taken for a feature to be delivered from scratch.

## Monitoring
### Azure Log Analytics
* IT Service Management Connector - allows you to connect Azure and a supported IT Service Management (ITSM) product such as ServiceNow. Ex: when there is an issue a ticket is generated.

### Java Tools
* Maven - build
* Gradle - build
* Chef - Config management, Infra as code
* Puppet - Config management
* Have to use Maven or Gradle for enabling SonarQube Analysis

## Infrastructure
## Virtual Machines
* Powershell Desired State Configuration (DSC) - Allows VMs to maintain a desired state.
* Azure Automation State Configuration allows you to write, manage and compile Powershell DSC
* Register-AzureRmAutomationDscNode - registers a VM node for DSC automation. 
  * ConfigurationMode:
    * ApplyOnly - Only applies config once
    * ApplyAndAutoCorrect - Auto corrects config
    * ApplyAndMonitor - Send alerts when config deviates
* Azure Policy Service can enforce rules over which VM sizes can be created. Applicable for other resources as well. This can be enforced via a gate in CD pipeline
![](/images/DevOps_CD_Gates.JPG)

## Azure App Center
* Initialize AppCenter with analytics about crashes - MSAppcenter.start(“{Your App Secret}”, withServices: [MSAnalytics.self, MSCrashes.self ])


