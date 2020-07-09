- [Messaging](#messaging)
  - [Storage Queues](#storage-queues)
  - [Azure Event Grid](#azure-event-grid)
  - [Azure Event Hub](#azure-event-hub)
  - [Azure Service Bus](#azure-service-bus)
    - [Queues](#queues)
    - [Topics](#topics)
- [Azure Storage](#azure-storage)
  - [Blobs](#blobs)
  - [Queues](#queues-1)
- [Azure App Service](#azure-app-service)
- [Azure Compute Solutions](#azure-compute-solutions)
  - [Virtual Machines](#virtual-machines)
  - [AKS](#aks)
  - [Azure Functions](#azure-functions)
  - [Azure Durable Functions](#azure-durable-functions)
    - [Function Chaining](#function-chaining)
    - [Fan-out/fan-in](#fan-outfan-in)
    - [Async HTTP APIs](#async-http-apis)
    - [Monitoring](#monitoring)
    - [Human Interaction](#human-interaction)
    - [Aggregator (stateful entities)](#aggregator-stateful-entities)
  - [Azure Container Registry](#azure-container-registry)
  - [Azure Container Instance](#azure-container-instance)
  - [Docker](#docker)
- [Databases & Caches](#databases--caches)
  - [Redis Cache](#redis-cache)
  - [Sql](#sql)
  - [Cosmos Db](#cosmos-db)
- [Integration Services](#integration-services)
  - [Azure Logic Apps](#azure-logic-apps)
  - [API Management](#api-management)
- [Security](#security)
  - [Azure Active Directory](#azure-active-directory)
  - [Azure Key Vault](#azure-key-vault)
- [Monitor, Troubleshoot and Optimize solutions](#monitor-troubleshoot-and-optimize-solutions)
  - [CDN](#cdn)
- [TODO](#todo)

# Messaging
![](images/comparision_message_services.png)
* `Events` - Notification that some state change has occurred. There is no expectation from sender. Light weight.
* `Messages` - Request from one system to another for some action to be taken.
  
## Storage Queues
* Support a max of `64 KB`
* Retreiving message from queue, takes it offline for 30 seconds. So, you need to delete after retreiving message if you don't want that message to be picked up by another client.

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
* To register event grid resource provider with Azure to use with other services - `az provider register --namespace Microsoft.EventGrid`. Not all services are registered by default.
  
## Azure Event Hub
* Big data pipeline - facilitates capture, retention and replay of telemetry and event stream data. At least once delivery
* Data sent to even hub can be transformed and stored by using any real-time analytics provider or `batching/storage` adapters.
* Connection string template - `Endpoint=sb://dummynamespace.servicebus.windows.net;EntityPath=EVENT_HUB_NAME;SharedAccessKeyName=SHARED_ACCESS_KEY_NAME;SharedAccessKey=SHARED_ACCESS_KEY`
* Send events to Event Hub:
```csharp
using Microsoft.Azure.EventHubs;

var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
await eventHubClient.CloseAsync();
```
* Receive events from Event Hub
  * Need to use EventProcessorHost .net calls which manages receiving events from hub. This class requires a storage account to store the persistent checkpoints.
  ```csharp
  public class SimpleEventProcessor : IEventProcessor
  {
    public Task CloseAsync(PartitionContext context, CloseReason reason)
    {
        Console.WriteLine($"Processor Shutting Down. Partition '{context.PartitionId}', Reason: '{reason}'.");
        return Task.CompletedTask;
    }

    public Task OpenAsync(PartitionContext context)
    {
        Console.WriteLine($"SimpleEventProcessor initialized. Partition: '{context.PartitionId}'");
        return Task.CompletedTask;
    }

    public Task ProcessErrorAsync(PartitionContext context, Exception error)
    {
        Console.WriteLine($"Error on Partition: {context.PartitionId}, Error: {error.Message}");
        return Task.CompletedTask;
    }

    public Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
    {
        foreach (var eventData in messages)
        {
            var data = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
            Console.WriteLine($"Message received. Partition: '{context.PartitionId}', Data: '{data}'");
        }

        return context.CheckpointAsync();
    }
  }
  ```
  ```csharp
  var eventProcessorHost = new EventProcessorHost(
                EventHubName,
                PartitionReceiver.DefaultConsumerGroupName,
                EventHubConnectionString,
                StorageConnectionString,
                StorageContainerName);
  // Registers the Event Processor Host and starts receiving messages
  await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();
  // Disposes of the Event Processor Host
  await eventProcessorHost.UnregisterEventProcessorAsync();
  ```
* `Partitions` - you can have between 2 and 32 partitions.
## Azure Service Bus
* Messages - transactions, ordering, duplicate detection and instantaneous consistency
### Queues
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

### Topics
* Filter conditions for subscriptions:
  * `Boolean filters` - `TrueFilter` and `FalseFilter` either cause all arriving messages (true) or none of the arriving messages (false) to be selected for the subscription
  * `SQL Filters` - holds SQL like conditional expressions that is evaluated in the broker against arriving messages. System properties should be prefixed by `sys.` and user defined properties should use `user.`
  * `Correlation Filter` - Unlike Boolean and SQL filters, this group is used to perform matching against one or more user and system properties in a very efficient way. Only works for checking equality. For system properties ContentType, MessageId, ReplyTo, ReplyToSessionId, SessionId, To, and CorrelationId can be assigned values to filter on.
* Retry Mechanisms built-in Service Bus SDK:
  * `RetryExponential` - Exponentially retries using specified intervals between back offs
  ```csharp
  var policy = new RetryExponential(
    minimumBackoff: TimeSpan.FromSeconds(10),
    maximumBackoff: TimeSpan.FromSeconds(30),
    maximumRetryCount: 3);
  ```
  * `NoRetry` - Does not retry
# Azure Storage
* SAS Tokens
  * `Account SAS` - These work at a storage account level and use the storage keys.
  * `Service SAS` - You can define a `stored access policy` on a specific container/file share/queue/table limiting the expiry of the policy and the access. This policy can then be used to create a SAS token. Advantage is that the permissions can be set per container and policy can be revoked independently of the storage key.
* Geo-redundant replication options:
  * `Geo-zone redundant storage (GZRS)` - data is replicated across 3 Azure availability zone in primary region and then replicated to secondary region
  * `Geo-redundant storage (GRS)` - data is replicated 3 times in locally redundant storage in primary region and then replicated to secondary region
  * Storage Retry Policies related to location (`LocationMode`):
    * `PrimaryOnly` - Uses primary location only
    * `PrimaryThenSecondary` - primary first and then secondary if it fails
    * `SecondaryOnly` - Uses secondary location only
    * `SecondaryThenPrimary` - secondary first and then primary if it fails
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
  * Using `Azure Active Directory` for security
    * Register app accessing storage in Azure AD.
    * Grant your registered app permissions to storage account. To use same user permissions as app use `user_impersonation` permission
    * This is a `delegated` type permission
* ```
  PUT https://company1.blob.core.windows.net/invoices/3000.pdf?comp=tier
  x-ms-access-tier: Archive
  ```
  This request changes a blob storage tier to Archive, thus making it offline.
* Update metadata with HTTP request - `PUT https://myaccount.blob.core.windows.net/mycontainer/myblob?comp=metadata`

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
## Queues
* Set visibility timeout - hide items in queue for specified time
  ```csharp
  CloudStorageAccount account = CloudStorageAccount.Parse(connectionString);
  CloudQueueClient client = account.CreateQueueClient();
  CloudQueue queue = client. GetQueueReference("orders");
  //De-queue 20 messages and set VisibilityTimeout for those messages to 5 mins. After 5 mins they will be visible again for de-queuing unless deleted.
  var messages = queue.GetMessages(20, TimeSpan.FromMinutes(5));
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
  * `Application Logging (file system)` - will capture traces and messages from app code. Can be accesses from `LogFiles/Application` sub folder of the root folder - once enabled.
  * `Web Server Logging` - captures HTTP requests using W3C extended log format.
  * `Detailed Error Messages` - enabling this captures HTTP errors that are returned from requests to web app. These are HTML files which can be downloaded from `LogFiles/DetailedErrors` sub folder of the root folder.
* Enabling CORS - `az webapp cors add --allowed-origins https://myapps.com --name MyWebApp --resource-group MyResourceGroup --subscription MySubscription`
* While deploying a `git repo` via `Kudu`, a `.deployment` file will let you override the deployment by allowing you to specify a project or folder to be deployed. You can also specify the custom deployment script to build and deploy your application.
* After authenticating with Facebook application code can retrieve the Facebook token from `X-MS-TOKEN-FACEBOOK-ACCESS-TOKEN` header 
* `Free` tier does not support custom domains
* When authenticating with external auth providers like google, facebook etc. - your redirect url should be something like `https://<yourdomain>/.auth/login<providername>/callback`
* Configure `swagger` - `services.AddSwaggerGen(c=>c.SwaggerDoc("v1",null));`
* Create a `WebJob` that will listen for requests until you terminate it
  ```csharp
  var config = new JobHostConfiguration();
  var host = new JobHost(config);
  host.RunAndBlock();
  ```

# Azure Compute Solutions
## Virtual Machines
* Creating a VM using disk encryption secured by a key from Azure Key Vault
  * `New-AzVM`
  * `Get-AzKeyVault`
  * `Set-AzVmDiskEncryptionExtension`
* Creating a custom image based on a VM:
  * Stop the VM
  * Set the status of VM to generalized - `Set-AzVm -ResourceGroupName $rgName -Name $vmName -Generalized`
  * Get the VM - `$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName`
  * Create the image config - `$image = New-AzImageConfig -Location $location -SourceVirtualMachineId $vm.Id`
  * Create the image - `New-AzImage -Image $image -ImageName $imageName -ResourceGroupName $rgName`
  * Image can be saved as a Blob in Storage
  * To create resources in a loop by concatenating index to differentiate resource names use `copyIndex(int offset)` in the ARM Template
  ```json
  {
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "resources": [
        {
           .
           .
            "name": "[concat('dev',copyIndex(1))]",
           .
           .
            "copy": {
                "name": "vmCopy",
                "count": "10"
            }
        }
    ],
    "outputs": {}
  }
  ```
* Open a port via CLI - `az vm open-port --port 80 --resource-group dev-rg --name dev1`
## AKS
* `Kubernetes CustomResourceDefinitions` - The CustomResourceDefinition API resource allows you to define custom resources. Defining a CRD object creates a new custom resource with a name and schema that you specify. The Kubernetes API serves and handles the storage of your custom resource.
* `KEDA` - Kubernetes Event Driven Architecture

## Azure Functions
* Authorization levels:
  * `anonymous` - No API key is required
  * `function` - function specific API key is required.
  * `admin` - master key is required.
* To change the path for API calls from the default path `/api/xxx` to `/xxx` we need to modify the `host.json` file and alter the `routePrefix` property apart from changing the route template/attribute in code.
* For functions with `Queue` triggers - it will automatically retry up to 5 times.

```csharp
public static class SimpleExample
{
    [FunctionName("QueueTrigger")]
    public static void Run([QueueTrigger("myqueue-items")] string myQueueItem, ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
    }
}
```

## [Azure Durable Functions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp)
* Function Types
  * `Orchestrator Functions`
    * describe how actions are executed and the order in which actions are executed
    * describe the orchestration in code
    * code must be deterministic
  * `Activity Functions`
    * are the basic unit of work in a durable function orchestration
    * are the functions and tasks that are orchestrated in the process
  * `Entity Functions`
    * define operations for reading and updating small pieces of state
    * can be called either from Client Functions or from Orchestrator Function
  * `Client Functions`
    * starts an orchestrator or entity function
* `Singleton for orchestrator functions`
  ```csharp
  public static async Task<HttpResponseMessage> Start([HttpTrigger(Route="{functionName}")] HttpRequestMessage req, [OrchestrationClient] DurableOrchestrationClient starter, string functionName){
    
  }
  ```


![](images/azure-durable-functions-20190427-16-638.jpg)
### Function Chaining
a sequence of functions executes in a specific order
```csharp
[FunctionName("Chaining")]
public static async Task<object> Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
  try
  {
      var x = await context.CallActivityAsync<object>("F1", null);
      var y = await context.CallActivityAsync<object>("F2", x);
      return  await context.CallActivityAsync<object>("F3", y);
  }
  catch (Exception)
  {
      // Error handling or compensation goes here.
  }
}
```
### Fan-out/fan-in
you execute multiple functions in parallel and then wait for all functions to finish
```csharp
[FunctionName("FanOutFanIn")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
  var parallelTasks = new List<Task<int>>();

  // Get a list of N work items to process in parallel.
  object[] workBatch = await context.CallActivityAsync<object[]>("F1", null);
  for (int i = 0; i < workBatch.Length; i++)
  {
      Task<int> task = context.CallActivityAsync<int>("F2", workBatch[i]);
      parallelTasks.Add(task);
  }

  await Task.WhenAll(parallelTasks);

  // Aggregate all N outputs and send the result to F3.
  int sum = parallelTasks.Sum(t => t.Result);
  await context.CallActivityAsync("F3", sum);
}
```
### Async HTTP APIs
* addresses the problem of coordinating the state of long-running operations with external clients. 
* exposes webhook HTTP APIs that query the orchestrator function status.
```
> curl -X POST https://myfunc.azurewebsites.net/orchestrators/DoWork -H "Content-Length: 0" -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec

{"id":"b79baf67f717453ca9e86c5da21e03ec", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec

{"runtimeStatus":"Running","lastUpdatedTime":"2019-03-16T21:20:47Z", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 200 OK
Content-Length: 175
Content-Type: application/json

{"runtimeStatus":"Completed","lastUpdatedTime":"2019-03-16T21:20:57Z", ...}
```
### Monitoring
refers to a flexible, recurring process in a workflow
```csharp
[FunctionName("MonitorJobStatus")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    int jobId = context.GetInput<int>();
    int pollingInterval = GetPollingInterval();
    DateTime expiryTime = GetExpiryTime();

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        if (jobStatus == "Completed")
        {
            // Perform an action when a condition is met.
            await context.CallActivityAsync("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }

    // Perform more work here, or let the orchestration end.
}
```
### Human Interaction
Includes human interaction/escalations in a workflow
```csharp
[FunctionName("ApprovalWorkflow")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval", null);
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(72);
        Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);

        Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
        {
            timeoutCts.Cancel();
            await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
        }
        else
        {
            await context.CallActivityAsync("Escalate", null);
        }
    }
}
```
### Aggregator (stateful entities)
aggregating event data over a period of time into a single, addressable entity
```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    int currentValue = ctx.GetState<int>();
    switch (ctx.OperationName.ToLowerInvariant())
    {
        case "add":
            int amount = ctx.GetInput<int>();
            ctx.SetState(currentValue + amount);
            break;
        case "reset":
            ctx.SetState(0);
            break;
        case "get":
            ctx.Return(currentValue);
            break;
    }
}
```

## Azure Container Registry
* If you assign a `service principal` to your registry, your application or service can use it for headless authentication.
* When working directly with ACR from a developer workstation you can authenticate using - `az acr login --name <acrName>`

## Azure Container Instance
* `az container create` - creates a new container instance

## Docker
* You can only have one `CMD` command in a `dockerfile`
* `docker tag` - adding a tag is required to push an image to a private registry like ACR

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
  * `Azure SQL Database Hyperscale` - A Hyperscale database is a database in SQL Database in the Hyperscale service tier that is backed by the Hyperscale scale-out storage technology. A Hyperscale database supports up to 100 TB of data and provides high throughput and performance, as well as rapid scaling to adapt to the workload requirements. You can also setup up read replicas to offload reads to additional compute replicas.\
  `az sql db create -g poc-app -s poc-db -n hyperscale1 -e Hyperscale --read-replicas 2` e = edition
  * Service Tiers:
    * `Hyperscale` - Up to 100 TB, scalable
    * `General purpose` - 8 TB max, low throughput
    * `Business Critical` - 4 TB max, intensive work loads
  * Command Behaviour:
  ```csharp
  command.ExecuteReader(CommandBehavior.CloseConnection);
  ```
    * `CloseConnection` - Closes connection immediately when DataReader object is closed
    * `Default` - May return multiple result sets
    * `KeyInfo` - returns column and primary key info
    * `SchemaOnly` - returns Schema only
    * `SequentialAccess` - For large binary values, rather than loading entire row enable loading data as a stream
    * `SingleResult` - query returns a single result
    * `SingleRow` - Returns a single row of the result set.
  * Add Sql database to elastic pool - `Set-AZSqlDatabase`
  * Sql `CommandType.TableDirect` - Specify a table name to run Sql statement against.
  * 

## Cosmos Db
* Implements APIs for SQL, Cassandra, MongoDB, Gremlin and Azure Table Storage
* Hierarchy - Account > Database > Container > Item
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
* You can move data from Mongo db into CosmosDb using `Azure Database Migration Service`
* Create a Cosmos account for Table API
  ```bash
  az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --capabilities EnableTable \
    --default-consistency-level Eventual \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --locations regionName='East US 2' failoverPriority=1 isZoneRedundant=False
  ```
* Create a Table API table
  ```bash
  az cosmosdb table create \
    -a $accountName \
    -g $resourceGroupName \
    -n $tableName \
    --throughput 400
  ```
# Integration Services

## Azure Logic Apps
* `Enterprise Integration Pack` - Connect several logic apps, edit B2B workflows.
* To secure Logic Apps with a Vnet we need to use `Integration Service Environment`. When a new ISE it is injected into your Azure VNet enabling direct access to private resources. Other benefits include - highly reliable performance, isolated private storage and predictable pricing.
* Custom Triggers:
  * `WebHook Trigger` - Logic App creates a custom endpoint for a web app to call to initiate the workflow
  * `Polling Trigger` - Polls a custom web app endpoint to determine when to start the workflow

## API Management
![](images/api_management_how_it_works.JPG)
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
* Following content must be added to the resources section of a json file for it to be recognized as a `Logic Apps Resource`:
  ```json
  {
    "type": "Microsoft.Logic/workflows"
  }
  ```
* Scopes - `Global > Product > API > Operation`
* Define a policy to restrict calls by bandwidth used:
  ```xml
  <policies>
    <inbound>
        <base />
        <quota calls="10000" bandwidth="40000" renewal-period="3600" />
    </inbound>
    <outbound>
        <base />
    </outbound>
  </policies>
  ```
  renewal-period is defined in `seconds` and bandwidth in `KB`
* Access restriction policies:
  * `check-header` policy - enforce that a request has specified HTTP header
  * `rate-limit` policy - limit call rate to a specified number
  * `rate-limit-by-key` policy - limit call rate by a key to a specified number. ex - limit calls per IP address for an hour.
  * `ip-filter` - only allow calls from specified ip address range
  * `quota` - limit by call volume or bandwidth by subscription. The renewal period cannot be less than 3600
  * `quota-by-key` - limit by call volume or bandwidth by a key - key can be ip address etc.
  * `validate-jwt` - enforces existence and validity of a jwt token
* `Versions` vs `Revisions`
  * Version - allows you to expose breaking changes, requires publishing.
  * Revision - allows you to add non-breaking changes, such as addition of operations. Users can access by using a different query string at the same endpoint.
* Create an instance - `New-AzApiManagement -ResourceGroupName` "myResourceGroup" `-Location` "West US" `-Name` "apim-name" `-Organization` "myOrganization" `-AdminEmail` "myEmail" `-Sku` "Developer"
  
# Security

## Azure Active Directory
* [Overview](https://www.youtube.com/watch?v=zjezqZPPOfc&list=PLLasX02E8BPBxGouWlJV-u-XZWOc2RkiX)
* `Microsoft Identity Platform` is the new name for Azure AD.
* Uses `OAuth 2.0` or `OpenID Connect`
* Each app needs to be registered with AAD in App Registrations where following info is collected:
  * Application Client ID that uniquely iDs your app
  * A Redirect URI
  * Few other scenario-specific values.
  * After app is registered, app communicates with the following endpoints
  ```
  https://login.microsoftonline.com/common/oauth2/v2.0/authorize
  https://login.microsoftonline.com/common/oauth2/v2.0/token
  ``` 
* OAuth vs OpenId Connect - OAuth is used for authorization and OpenID Connect (OIDC) is used for authentication. OpenID Connect is built on top of OAuth 2.0, so the terminology and flow are similar between the two.
* Security Tokens
  * Access Tokens and Refresh tokens are issued as part of OAuth flow
  * IDTokens are sent as part of the OpenID connect flow
* Implicit flow is now the recommended flow
![](/images/active-directory-oauth-code-spa.png)
* WebApps can use `OpenID Connect`
![](images/active-directory-openid-connect-spa.png)
* WebAPIs using access tokens
![](images/webapi-auth-access-token.png)
* `Service Principal` object:
  * When you register an app with Azure AD and its given permissions to access resources in a tenant the Service Principal object is created. It defines the access policy and permissions for the user/application in the Azure AD tenant.
* Scopes and permissions
  * Any web-hosted resource that integrates with AAD has a resource identifier or App ID URI which can define a set of permissions that can be used to define a set of permissions fir dividing the functionality of a resource. In OAuth 2.0 these types of permissions are called `scopes`. A permission is represented in the AAD as a string value. Ex - Read a user's calendar by using Calendars.Read
* Permission Types
  * `Delegated permissions` - are used by apps that have a signed-in user present
  * `Application permissions` - are used by apps that run without a signed-in user present; for example, apps that run as background services or daemons.
* [Authentication flows and app scenarios](https://docs.microsoft.com/en-us/azure/active-directory/develop/authentication-flows-app-scenarios)
* Multi-tenant applications will allow users from outside the application's home tenant to login to the app. Example login via Google account.
  * `user.read` scope allows a user to sign in and read his user profile from an app - this is a `delegated` type permission
* Usage via Azure CLI
  ```bash
  # create AD group
  az ad group create --display-name appdev --mail-nickname appdev

  # Assign a role 
  az role assignment create \
  --assignee $APPDEV_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
  ```
* `Enterprise application vs App Registration`:
  * Enterprise application - allows you to integrate other applications for use with Azure AD
  * App Registration - Allows your app to authenticate using Azure AD.
* `Public` vs `Confidential clients`
  * `Public Client` - are apps that run on devices or desktop computers or in a web browser. They're not trusted to safely keep application secrets, so they only access web APIs on behalf of the user.
  * `Confidential client` - are apps that run on servers (web apps, web API apps, or even service/daemon apps).
* [MSAL initializing client apps](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-net-initializing-client-applications)
* `Enabling Multi Factor Authentication`:
  * Needs Azure AD premium
  * Create a conditional access policy
*   
## Azure Key Vault
* Encrypting using key - `client.EncryptAsync("https://companyA.vault.azure.net/encryptionKey", "RSAOAEP256", securityQuestions);`
* Update attributes of a key in key vault - `client.UpdateKeyAsync("https://companyA.vault.azure.net/keys/encryptionKey", new[]{"encrypt", "decrypt"})`
* Update attributes of a secret in key vault - `client.SetSecretAsync("https://companyA.vault.azure.net/keys/encryptionKey", "operations, "encrypt,decrypt")`
* Get a key from key vault - `client.GetKeyAsync("https://companyA.vault.azure.net", "encryptionKey");`

# Monitor, Troubleshoot and Optimize solutions
## CDN
* Purging CDN in portal make it immediately get the contents from the site.
* To add and endpoint:
  * Set Origin type to `Custom Origin` if you are trying to cache from your own custom domain. If you choose other options Azure will automatically set the origin name to <your_resource_name>.<resource_type>.core.windows.net. Ex - if you choose storage - origin will be set to mystorageaccount.blob.core.windows.net 
# TODO
* Event Hub - partitions
* Event Grid
