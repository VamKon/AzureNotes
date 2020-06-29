- [Messaging](#messaging)
  - [Storage Queues](#storage-queues)
  - [Azure Event Grid](#azure-event-grid)
  - [Azure Event Hub](#azure-event-hub)
  - [Azure Service Bus](#azure-service-bus)
- [Azure Storage](#azure-storage)
  - [Blobs](#blobs)
  - [Table](#table)
- [Azure App Service](#azure-app-service)
- [AKS](#aks)
- [Databases & Caches](#databases--caches)
  - [Redis Cache](#redis-cache)
  - [Sql](#sql)
  - [Cosmos Db](#cosmos-db)
- [Integration Services](#integration-services)
  - [Azure Logic Apps](#azure-logic-apps)
  - [API Management](#api-management)
- [Virtual Machines](#virtual-machines)

# Messaging
![](images/comparision_message_services.png)
* `Events` - Notification that some state change has occurred. There is no expectation from sender. Light weight.
* `Messages` - Request from one system to another for some action to be taken.
  
## Storage Queues
* Support a max of `64 KB`

## Azure Event Grid
* Supports up to 64 KB messages
* Event Grid `Topic` provides and endpoint where the source sends events.
  * `System Topics` are those that are provided by other Azure services such as Service Bus, Azure Storage etc.
  * `Custom Topics` are application and third party topics. Send requests to url like `https://exampletopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01`
* Event Grid `Subscription` forwards the messages from a topic to a specified endpoint for handling the event.
* `Event Handlers` - is the place where an event is sent. Event Handler takes further action to process the event.
* At least once delivery.
* Event Schema
  ```csharp
  [
  {
    "topic": string,
    "subject": string,
    "id": string,
    "eventType": string,
    "eventTime": string,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string,
    "metadataVersion": string
  }
  ]
  ```
* `Event Filtering` for subscriptions
  * `Event Type Filtering` - You can decide to send only certain event types to your endpoint. For example, you can get notified of updates to your resources, but not notified for other operations like deletions.
  ```json
  "filter": {
  "includedEventTypes": [
    "Microsoft.Resources.ResourceWriteFailure",
    "Microsoft.Resources.ResourceWriteSuccess"
    ]
  }
  ```
  * `Subject Filtering` - For simple filtering by subject, specify a starting or ending value for the subject.
  ```json
  "filter": {
  "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
  "subjectEndsWith": ".jpg"
  }
  ```
  * `Advanced Filtering` - To filter by values in the data fields and specify the comparison operator
  ```json
  "advancedFilters": [
    {
        "operatorType": "StringContains",
        "key": "Subject",
        "values": [
            "/providers/microsoft.devtestlab/",
            "/providers/Microsoft.Compute/virtualMachines/"
        ]
    }
  ]
  ```

## Azure Event Hub
* Big data pipeline - facilitates capture, retention and replay of telemetry and event stream data. At least once delivery

## Azure Service Bus
* Messages - transactions, ordering, duplicate detection and instantaneous consistency
* Queues support a max size of `256 KB`
* Code to send messages:
  ```csharp
  queueClient = new QueueClient(ServiceBusConnectionString, QueueName);
  await queueClient.SendAsync(message);
  await queueClient.CloseAsync();
  ```
* Received messages:
  ```csharp
  queueClient = new QueueClient(ServiceBusConnectionString, QueueName);
  queueClient.RegisterMessageHandler(() => {
    await queueClient.CompleteAsync(message.SystemProperties.LockToken);
  }), messageHandlerOptions);
  await queueClient.CloseAsync();
  ```

# Azure Storage
## Blobs
* V2 accounts support `Azure Storage Events` - pushes events like blob creation/deletion to Azure Event Grid for downstream processing etc.
* `Change Feed Support` - provide transaction logs of all changes to blobs which are ordered, durable and immutable. Apps can read these logs too.
* Access Tiers
  * Hot - optimized for frequently accessed data
  * Cool - infrequently accessed data and stored for at least 30 days
  * Archive - rarely accessed data and stored for at least 180 days. It's considered offline and can't be read or modified. To read data from Archive tier
    * Rehydrate an archived blob to an online tier - can take hours to complete
    * Copy an archived blob to an online tier
* Copy files to storage recursively \
  `azcopy copy "C:\local\path" "https://account.blob.core.windows.net/mycontainer1/?sv=2018-03-28&ss=bjqt&srt=sco&sp=rwddgcup&se=2019-05-01T05:01:17Z&st=2019-04-30T21:01:17Z&spr=https&sig=MGCXiyEzbtttkr3ewJIh2AR8KrghSy1DGM9ovN734bQF4%3D" --recursive=true`
* code \
  ```csharp
  BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString, "container_name");
  BlobClient blob = blobServiceClient.GetBlobClient("file_name");
  BlobDownloadInfo download = blob.Download();
  using (FileStream file = File.OpenWrite("file_downloadPath"))
  {
      download.Content.CopyTo(file);
  }
  ```
  * Lease a blob - `http://127.0.0.1:10000/devstoreaccount1/mycontainer/myblob?comp=lease` gives exclusive write access.
  * If you enable `soft delete` on blobs, both blobs and snapshots can be recovered if deleted.
## Table
* Operating on tables via code:
  ```csharp
  CloudStorageAccount storage = CloudStorageAccount.Parse(conn_string);
  CloudTableClient table_client = storage.CreateCloudTableClient();
  CloudTable table = table_client.GetTableReference("Customer");
  TableOperation retreiveOperation = TableOperation.Retreive<CustomerEntity>("[PartitionKey], [RowKey]");
  TableResult retreivedResult = table.Execute(retreiveOperation);
  ```
* `TableBatchOperation` - collection of table operations, can contain up to 100.
  ```csharp
  TableBatchOperation batch = new TableBatchOperation();
  batch.Insert(...data);
  table.ExecuteBatch(batch);
  ```
* Filtering on PartitionKey and RowKey properties
  ```html
  https://myaccount.table.core.windows.net/Customers(PartitionKey='MyPartition',RowKey='MyRowKey1')
  ```
  ```csharp
  TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Dave");
  ```

# Azure App Service
* If `WEBSITES_ENABLE_APP_SERVICE_STORAGE` setting is unspecified or set to true, the /home/ directory will be shared across scale instances, and files written will persist across restarts. Explicitly setting WEBSITES_ENABLE_APP_SERVICE_STORAGE to false will disable the mount.
* While using an `Application Gateway (WAF)` - for end to end TLS, trusted Azure Services such as Azure Web Apps do not require adding any additional SSL certs. SSL certs are still required for TLS termination at app gateway.
* You are not charged extra for `Deployment slots`
* `WebJobs` are a feature of Azure App Service that enables you to run a program or script in the same instance of the web app, there is no extra charge.
* Using docker image:\
`az webapp config container set --name` [app-name] `--resource-group` [myResourceGroup] `--docker-custom-image-name` [azure-container-registry-name].azurecr.io/[mydockerimage]:v1.0.0 `--docker-registry-server-url` https://[azure-container-registry-name].azurecr.io `--docker-registry-server-user` [registry-username] `--docker-registry-server-password` [password]
* Deploy code from public github repo:\
  `az webapp deployment source config --name` webappname `--resource-group` myResourceGroup `--repo-url` gitrepourl `--branch` master `--manual-integration` \
  manual-integration disables automatic sync 
* App Service Plans:
  * Free, Shared - run on same VM as other app service plans, cannot scale
  * Basic, Standard, Premium and Premium V2 - runs on dedicated VMs. Higher the tier more the scaling. Deployment slots are only available on Standard plan and up
  * Isolated - dedicated VMs, dedicated VNets, provides network and compute isolation, max scale out capabilities.
* `Always On` can be enabled to keep app always loaded even when there is no traffic. Available in `Basic` service plan and above.
* You can scale based on a custom metric from App Insights
* To use Application Insights - `Always On` should be enabled.
* Logging:
  * to stream logs - `az webapp log tail --name appname --resource-group myResourceGroup`
  * filter errors in logs - `az webapp log tail --name appname --resource-group myResourceGroup --filter Error`
  * filter specific log types - `az webapp log tail --name appname --resource-group myResourceGroup --path http`
  * enable docker container logs - `az webapp log config --docker-container-logging filesystem --name MyWebapp --resource-group MyResourceGroup`

# AKS
* `Kubernetes CustomResourceDefinitions` - The CustomResourceDefinition API resource allows you to define custom resources. Defining a CRD object creates a new custom resource with a name and schema that you specify. The Kubernetes API serves and handles the storage of your custom resource.
* `KEDA` - Kubernetes Event Driven Architecture

# Databases & Caches
## Redis Cache
```csharp
//Establish connection
IDatabase cache = Connection.GetDatabase();
//adding to cache
cache.StringSet("key", "value");
//retrieve object from cache
cache.StringGet("key");
//Invalidating cache
cache.KeyDelete("key");
```
## Sql
* Protecting sensitive data
  * `Dynamic Data Masking` - Masks specified data in columns in the UI/presentation layer - queries from SSMS, application data etc. but does not actually change data in the database
  * `Always Encrypted` - Data in the database is encrypted at rest, during movement and while data is in use. After configuring only client apps or app servers that have access to keys can access plaintext data. App uses a driver to decrypt sensitive data for display.

## Cosmos Db
* [Consistency Levels](https://docs.microsoft.com/en-us/azure/cosmos-db/consistency-levels):
  * `Strong`: The reads are guaranteed to return the most recent committed version of an item
  * `Bounded Staleness`: The reads might lag behind writes by at most "K" versions (that is, "updates") of an item or by "T" time interval, whichever is reached first. You have to set these up individually.
  * `Session`: Within a single client session reads are guaranteed to honor the consistent-prefix. Most widely chosen
  * `Consistent Prefix`: guarantees that reads never see out-of-order writes
  * `Eventual`: There's no ordering guarantee for reads. In the absence of any further writes, the replicas eventually converge.
* Cosmos Db with regional failover\
  `az cosmosdb update --name "abc" --resource-group "abc" --locations "South central US"=0 "North Central US"=1 "East US" = 2`
* Azure Cosmos DB provides language-integrated, transactional execution of JavaScript that lets you write stored procedures, triggers, and user-defined functions (UDFs).
  * Stored procs:
  ```javascript
  var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
  }
  ```
  The context object provides access to all operations that can be performed in Azure Cosmos DB, as well as access to the request and response objects.
  * Triggers - Pre-triggers are executed before modifying a database item and post-triggers are executed after modifying a database item. Pre-triggers cannot have any input parameters, and the request message body contains the item to be created in JSON format
  * Pre-Trigger
  ```javascript
  function validateToDoItemTimestamp() {
    var context = getContext();
    var request = context.getRequest();

    // item to be created in the current operation
    var itemToCreate = request.getBody();

    // validate properties
    if (!("timestamp" in itemToCreate)) {
        var ts = new Date();
        itemToCreate["timestamp"] = ts.getTime();
    }

    // update the item that will be created
    request.setBody(itemToCreate);
  }
  ```
  * Post-Trigger
  ```javascript
  function updateMetadata() {
  var context = getContext();
  var container = context.getCollection();
  var response = context.getResponse();

  // item that was created
  var createdItem = response.getBody();

  // query for metadata document
  var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
  //logic to update data here
  return;
  }
  ```
  * User defined function:
  ```javascript
  function tax(income) {
    if(income == undefined)
        throw 'no input';

    if (income < 1000)
        return income * 0.1;
    else if (income < 10000)
        return income * 0.2;
    else
        return income * 0.4;
    }
  ```

# Integration Services

## Azure Logic Apps
* `Enterprise Integration Pack` - Connect several logic apps, edit B2B workflows.

## API Management
* Policies - collection of statements that are executed sequentially on the request or response of an API
  ```xml
  <policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
  </policies>
  ```

# Virtual Machines
* Creating a VM using disk encryption secured by a key from Azure Key Vault
  * `New-AzVM`
  * `Get-AzKeyVault`
  * `Set-AzVmDiskEncryptionExtension`