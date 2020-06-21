- [Messaging](#messaging)
  - [ServiceBus](#servicebus)
  - [Storage Queues](#storage-queues)
- [Azure Storage](#azure-storage)
  - [Blobs](#blobs)
  - [Table](#table)
- [Azure App Service](#azure-app-service)
- [AKS](#aks)
- [Databases & Caches](#databases--caches)
  - [Redis Cache](#redis-cache)
  - [Cosmos Db](#cosmos-db)

# Messaging
## ServiceBus
* Queues support a max size of `256 KB`
## Storage Queues
* Support a max of `64 KB`

# Azure Storage
## Blobs
* V2 accounts support `Azure Storage Events` - pushes events like blob creation/deletion to Azure Event Grid for downstream processing etc.
* `Change Feed Support` - provide transaction logs of all changes to blobs which are ordered, durable and immutable. Apps can read these logs too.
## Table
* Operating on tables via code:
  ```csharp
  CloudStorageAccount storage = CloudStorageAccount.Parse(conn_string);
  CloudTableClient table_client = storage.CreateCloudTableClient();
  CloudTable table = table_client.GetTableReference("Customer");
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


# AKS
* `Kubernetes CustomResourceDefinitions` - The CustomResourceDefinition API resource allows you to define custom resources. Defining a CRD object creates a new custom resource with a name and schema that you specify. The Kubernetes API serves and handles the storage of your custom resource.
* `KEDA` - Kubernetes Event Driven Architecture

# Databases & Caches
## Redis Cache
```csharp
//Establish connection
IDatabase cache = Connection.GetDatabase();
//Invalidating cache
cache.KeyDelete(string)
```

## Cosmos Db
* [Consistency Levels](https://docs.microsoft.com/en-us/azure/cosmos-db/consistency-levels):
  * `Strong`: The reads are guaranteed to return the most recent committed version of an item
  * `Bounded Staleness`: The reads might lag behind writes by at most "K" versions (that is, "updates") of an item or by "T" time interval, whichever is reached first. You have to set these up individually.
  * `Session`: Within a single client session reads are guaranteed to honor the consistent-prefix. Most widely chosen
  * `Consistent Prefix`: guarantees that reads never see out-of-order writes
  * `Eventual`: There's no ordering guarantee for reads. In the absence of any further writes, the replicas eventually converge.
