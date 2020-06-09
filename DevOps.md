# DevOps

- [DevOps](#devops)
  - [Deployments](#deployments)
    - [ARM Templates](#arm-templates)
    - [Azure CLI](#azure-cli)
  - [Azure Key Vault](#azure-key-vault)
  - [AKS](#aks)
  - [ACR](#acr)
  - [Helm](#helm)
  - [Docker](#docker)
  - [Azure Devops](#azure-devops)
    - [Artifacts](#artifacts)
    - [CI](#ci)
    - [CD](#cd)
    - [Jenkins](#jenkins)
    - [Repository](#repository)
    - [Agile](#agile)
    - [Testing](#testing)
  - [Monitoring](#monitoring)
  - [Others](#others)
    - [Azure Automation](#azure-automation)
    - [Virtual Machines Desired State Configuration](#virtual-machines-desired-state-configuration)
    - [Azure App Center](#azure-app-center)
    - [Azure Service Fabric](#azure-service-fabric)
    - [Other Tools](#other-tools)

## Deployments
### ARM Templates
* To deploy complex solutions, you can break a template into many templates, and deploy these templates through a main template.
* Use `CustomScriptExtension` to run custom scripts on a VM when it starts.
* `az deployment [group] create`:
  * group - RG
  * sub - subscription. Running az deployment without specifying sub runs the ARM template at subscription level.
  * mg - management group
  * tenant - tenant
* Deployment Modes:
  * `Complete` - In complete mode, Resource Manager deletes resources that exist in the resource group but aren't specified in the template.
  * `Incremental` - Default mode. Leaves unchanged resources that exist in the resource group but aren't specified in the template. Resources in the template are added to the resource group.
### Azure CLI
* To setup a connection string using CLI - `az webapp config connection-string set -g MyResourceGroup -n MyUniqueApp -t mysql \
    --settings mysql1='Server=myServer;Database=myDB;Uid=myUser;Pwd=myPwd;'`

## Azure Key Vault
* Need to enable Azure Key Vault for template deployment so that ARM Template can read secrets from KeyVault (`enabledForTemplateDeployment` property)
* Accessing a secret from ARM template:
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "adminLogin": {
        "value": "exampleadmin"
      },
      "adminPassword": {
        "reference": {
          "keyVault": {
          "id": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"
          },
          "secretName": "ExamplePassword"
        }
      },
      "sqlServerName": {
        "value": "<your-server-name>"
      }
      }
}
```

## AKS
* Scaling nodes = `az aks scale -g myResourceGroup -n myAKSCluster --node-count 3`
* To allow an AKS cluster to interact with other Azure resources, an `Azure Active Directory service principal` is used.
* `Roles` are used to assign permissions to users for a namespalce, `ClusterRoles` are used to assign permissions for a cluster.
* *Role bindings* are used to assign *roles* for a given *namespace*. If you need to bind roles across the entire cluster, or cluster resources outside a given namespace, you can instead use `ClusterRoleBindings`
* `Docker compose` enables you to bring up multiple containers and run tests. You can use a `docker-compose.yml` file to define two containers that need to work together.
* Scale pods manually - `kubectl scale --replicas=n deployment/[podname]`
* The `kubelet` is the primary “node agent” that runs on each node. The kubelet works in terms of a PodSpec. A PodSpec is a YAML or JSON object that describes a pod. The kubelet takes a set of PodSpecs that are provided through various mechanisms (primarily through the apiserver) and ensures that the containers described in those PodSpecs are running and healthy.


## ACR
* Docker + ACR:
  * Log in to a registry - `az acr login --name myregistry` or `docker login myregistry.azurecr.io` if a service principal has been assigned to your registry
  * Create an alias of the image to avoid cluttering the root - `docker tag nginx myregistry.azurecr.io/samples/nginx`. nginx image is copied to samples folder
  * Push image to ACR - `docker push myregistry.azurecr.io/samples/nginx`
  * Pull - `docker pull myregistry.azurecr.io/samples/nginx`
  * Run the image - `docker run -it --rm -p 8080:80 myregistry.azurecr.io/samples/nginx`
* `az acr build` - build an image
* `acr push` role allows you to push images.
* `Docker Trusted Registry` scans container images automatically.

## Helm
* Helm init - install tiller on AKS
* Helm install - install helm charts on AKS
* Need to use a `service account` to install helm on an AKS cluster using RBAC.

## Docker
* `RUN` - Executes commands in a new layer and creates a new image. Use as few Run statements as possible, combine them.
* `CMD` - allows you to set a default command, which will be executed only when you run the container without specifying a command.
* `ENTRYPOINT` - allows you to configure a container that will run as an executable. The command and params are not ignored when docker container runs with command line params.
* Docker yaml task required param - `Command`. Options are buildAndPush, build, push, login and logout

## Azure Devops
* Test & Feedback extension
  * `Stakeholder` access to Azure DevOps allows users to use the `Test & Feedback extension` to request for and respond to provide feedback.
  * `Basic` access allows users to use the extension to perform exploratory testing
* Security of Agent Pools
  * `Reader` - View agents and agent pools
  * `Service Account` - Use org agent pools to create project agent pools
  * `Administrator` - Create new agent pools
* Gathering Feedback for work items
  * `Stakeholder` - provide or review feedback
  * `Basic` - Request feedback
* Licensing
  * Azure DevOps comes with 5 free `Basic` licenses giving access to all feature except for Test Manager.
  * `Basic + Test plans` - Basic features plus access to Test Managers (mostly for QA)
  * `Stakeholder` - Access to boards etc. Unlimited free licenses for this. No access to code repos.
* `win1803` - Windows Server Core 1803 image (for running windows containers)
* Service Connections are part of project settings not organization settings
* Azure Pipeline Library
  * Secure Files - se the Secure Files library to store files such as signing certificates, Apple Provisioning Profiles, Android Keystore files, and SSH keys on the server without having to commit them to your source repository. Pipelines can download and use these files using the `Download secure file` task.
### Artifacts
* Public Feeds - you need to have a public project to create a public feed. You cannot convert an existing project scoped feed to a public feed.
* .npmrc (npm config) file:
  * in user’s home folder will contain credentials for all of the registries needed. ($Home = linux, $env.Home = win)
  * In project root beside project.json will contain registry info
  * ![](/images/Artifacts_FeedPermissionsOverview.JPG)
* `nuget push -Source <SourceName> -ApiKey az <PackagePath exp:(.\Get-Hello.1.0.0.nupkg)>`

### CI
* Java - To deploy complex solutions, you can break a template into many templates, and deploy these templates through a main template.
* Checking the “Batch” option in the triggers tab of a build will reduce the number of times the CI builds will run if multiple check ins are being made. when a pipeline is running, the system waits until the run is completed, then starts another run with all changes that have not yet been built.
* If you are using *Cobetura* or *JaCoCo* to obtain code coverage, you have to use `Publish Code Coverage Results task` to publish code coverage results. You don't need to use this task if you use these tasks in your CI pipeline - VS Test, tests in .net core, Ant, Maven, Gulp, Grunt.
* Setting the value to zero for timeouts in pipelines results in:
  * Self-hosted agents = forever
  * Microsoft-hosted agents with a public project and public repo = 360 mins (6 hrs)
  * Microsoft-hosted agents with a private project and private repo = 60 mins
* Microsoft-hosted agents create a new instance each time, so there is not caching etc. which will be slower than self-hosted agents.
* Costs for parallel jobs - self-hosted agent has a free tier which can run unlimited parallel jobs on self-hosted VMs for *public projects*
* Use `Azure DevTest Labs` in pipelines to build code from non-supported languages. Using the `Azure DevTest Labs Tasks Extension` you can integrate setting up VMs with custom images, tools etc. to use them for build and deployment and tear them down when done.
* `Container Jobs` - CI/CD jobs run inside a container on a host VM. That way we don't need to have dedicated VMs with specific configs. They are only available in yaml pipelines. 
  * For Linux based containers following are required:
    * Bash
    * glibc-based
    * Can run Node.js
    * Does not define an ENTRYPOINT
    * User has access to groupadd and other privileges commands without sudo
  * And on the host:
    * Docker must be installed
    * Agent must have permission to access the Docker daemon
* YAML pipeline schema:
  * If you have a single stage, you can omit the stages keyword and directly specify the jobs keyword
  * If you have a single stage and a single job, you can omit the stages and jobs keywords and directly specify the steps keyword
```
name: string  # build numbering format
resources:
  pipelines: [ pipelineResource ]
  containers: [ containerResource ]
  repositories: [ repositoryResource ]
variables: # several syntaxes, see specific section
trigger: trigger
pr: pr
stages: [ stage | templateReference ]
```
* [Service Containers](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/service-containers?view=azure-devops&tabs=yaml) - If your pipeline requires the support of one or more services, in many cases you'll want to create, connect to, and clean up each service on a per-job basis. For instance, a pipeline may run integration tests that require access to a database and a memory cache. The database and memory cache need to be freshly created for each job in the pipeline.
  * Use `script` task to run commands in container
  
### CD
* Pre/Post Deployment Approvals - Manual Intervention
* Pre/Post Deployment Gates - Automation
* For Blue-Green deployments using weighted routing method use Azure Traffic Manager
![](images/azuretrafficmanager_weightedrouting.png)
* Use `Deployment Groups` in pipelines to deploy to a group of servers. Deployment Groups also allow you to deploy to a subset of servers to perform a safe rolling deployment. Servers can be a combination of cloud/on-prem. Use `Deployment Group Job` in pipeline for this.
* `App Center Distribute` task - Distribute app builds to testers and users throught app center
* To restrict a variable to be used for a specific stage in a release, you have to set the `scope` for the variable to the appropriate stage.

### Jenkins
* To allow Jenkins access to ACR, you need to create a service principal in Azure. In Jenkins you need to create a credential resource for the ACR service principal.
* Install Azure Storage Plugin for Jenkins to store build output in Azure Storage.
* To use code from Git repo in Azure Devops:
  * Add Git Url
  * Add a Personal Access Token from DevOps into Jenkins
* To kick off a CI build in Jenkins from a repo managed in Azure DevOps, we need to create a service hook in Azure DevOps.

### Repository
* User “Choose import from Git repository” to import an existing repo in GitHub to Azure DevOps
* The Azure Boards app for GitHub is the preferred method for integrating Azure Boards with GitHub. By connecting you can support linking between GitHub commits and pull requests to work items in Azure.
* Adding a repo to Azure boards - Add it from GitHub Connections in Azure Devops, project settings.
* With TFVC you can set permissions at a file level, which is not possible with git
* Locking a branch does not prevent PRs from being completed
* Merge Types:
  * `Merge (no fast-forward)`: Default. All the individual commits in the pull request branch are preserved as-is, and a new merge commit is created to unite the master branch and the pull request branch.
  * `Squash commit`: Squashing will take the tree that’s produced in a merge and creates a single new commit with those repository contents.
  * `Rebase`: Rebase will take each individual commit in the pull request and cherry-pick them onto the master branch.
  * `Semi-linear` or `Rebase and fast-forward`: it’s a mix of rebase and a merge. First, the commits in the pull request are rebased on top of the master branch. Then those rebased pull requests are merged into master branch.
* oAuth is the preferred auth method from AzureDevOps to Git.
* For hosting Java packages you can use Azure Devops, Nexus or Artifactory

### Agile
* Cumulative Flow Diagram - Count of work items (over time) for each column of a kanban board
* Cycle Time - Time it takes for your team to complete work items once they begin actively working on them
* Lead Time - Time taken for a feature to be delivered from scratch.
* Burndown - Trend of remaining work across multiple teams and multiple sprints
* Burnup - Trend of completed work across multiple teams and multiple sprints
* Velocity - How much work your team can complete during a sprint
* **[Process](https://docs.microsoft.com/en-us/azure/devops/boards/work-items/guidance/choose-process?view=azure-devops&tabs=basic-process#basic-agile-scrum-and-cmmi)**:
  * `Basic`: simplest model that uses Issues, Tasks, and Epics to track work. Taks support remaining work
  * `Agile`: tracks development and test activities separately. This process works great if you want to track user stories and (optionally) bugs on the Kanban board, or track bugs and tasks on the taskboard. Tasks support tracking original work, completed work and remaining work. 
  * `Scrum`: works great if you want to track product backlog items (PBIs) and bugs on the Kanban board, or break PBIs and bugs down into tasks on the taskboard. Tasks support tracking remaining work only.
  * `CMMI`: With this process, you can track requirements, change requests, risks, and reviews - more formal project methods. Tasks support tracking original work, completed work and remaining work.
* `Kanban board` allows you to allows a user to add,view and interact directly. Allows you to make changes to test cases directly.
* Adding AB#[Work Item ID] in the commit message will link commit to work item.

### Testing
* *Test plans* - used to group together test suites and test cases
* *Test suites* - group test cases
* *Test cases* - individual tests, steps for test
* To reduce costs for cloud-based load testing - limit the number of `virtual user minutes` that the testing uses, because you are charged for the number of VUMs used.

## Monitoring
* IT Service Management Connector - allows you to connect Azure and a supported IT Service Management (ITSM) product such as ServiceNow. Ex: when there is an issue a ticket is generated.
* `Smart Groups` allow you to group similar notifications for less clutter.
* Do not use Azure Monitor for monitoring VMs as VMs will generate a lot of events.

## Others
### Azure Automation
* You can store Azure Automation Runbooks in Source Control. GitHub, Azure Repos are supported.
* To run Ansible scripts from inside Azure network - its best to use `Azure Cloud Shell` as support for Ansible is built in.
### Virtual Machines Desired State Configuration
* Powershell Desired State Configuration (DSC) - Allows VMs to maintain a desired state.
* Azure Automation State Configuration allows you to write, manage and compile Powershell DSC
* `Import-AzAutomationDscConfiguration -SourcePath 'C:\DscConfigs\TestConfig.ps1' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Published` - Import/upload a DSC config file
* `Start-AzAutomationDscCompilationJob -ConfigurationName 'TestConfig' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount'` - compile the config
* `Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm'` - Registers a VM node.
* Register-AzureRmAutomationDscNode - registers a VM node for DSC automation. 
  * ConfigurationMode:
    * ApplyOnly - Only applies config once
    * ApplyAndAutoCorrect - Auto corrects config
    * ApplyAndMonitor - Send alerts when config deviates
* Azure Policy Service can enforce rules over which VM sizes can be created. Applicable for other resources as well. This can be enforced via a gate in CD pipeline
![](/images/DevOps_CD_Gates.JPG)

### Azure App Center
* Initialize AppCenter with analytics about crashes - `MSAppcenter.start(“{Your App Secret}”, withServices: [MSAnalytics.self, MSCrashes.self ])`
* Certificate must be in a `.p12` format
* Distribution Groups
  * Private - default, Only people invited via email can access releases available to this group. To use device registration for Apple devices, you are limited to use of non-public, non-shared distribution groups.
  * Public - enables unauthenticated installs from public links.
  * Shared - shared across multiple apps in a single organization, eliminate the need to replicate distribution groups across multiple apps
* To send App Center analytics to App Insights:
  * Link Azure Subscription with App Center
  * Export App Center Analytics to Application Insights
* IOS apps have to be signed, a provision profile and register a test device in Apple Developer portal for testing. Apps for android don't need to be signed for testing.

### Azure Service Fabric
* For deployment of an application using Azure Pipelines to Azure Service Fabric, you would need to have the cluster endpoint and the Service Certificate Thumbprint.
* Cluster Resource Manager does not perform any type of load balancing. So, to ensure microservices run efficienctly, you have to perform load balancing.

### Other Tools
* Maven - Java build
* Gradle - Java build
* Chef - Config management, Infra as code
* Puppet - Config management, Infra as code
* Have to use Maven or Gradle for enabling SonarQube Analysis
* Bamboo - CI/CD tool from Altassian
* PMD, FindBugs - Java static code analysis. Can be setup using Maven or Gradle tasks
* You can enable and `execution strategy` in Entity Framework configuration to setup retry logic for transient failures with the database.
* For integration with `Trello` from Azure Devops
  * Create a service hook to Trello in Devops
  * Allow Azure Devops permissions in Trello
* `Azure Site Recovery` will enable you to replicate your VM from a private data center to Azure wihout taking the application offline.


