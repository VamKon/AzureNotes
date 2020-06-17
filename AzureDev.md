- [Messaging](#messaging)
  - [ServiceBus](#servicebus)
  - [Storage Queues](#storage-queues)
- [Azure Storage](#azure-storage)
  - [Blobs](#blobs)
- [Azure App Service](#azure-app-service)

# Messaging
## ServiceBus
* Queues support a max size of `256 KB`
## Storage Queues
* Support a max of `64 KB`

# Azure Storage
## Blobs
* V2 accounts support `Azure Storage Events` - pushes events like blob creation/deletion to Azure Event Grid for downstream processing etc.
* `Change Feed Support` - provide transaction logs of all changes to blobs which are ordered, durable and immutable. Apps can read these logs too.

# Azure App Service
* If `WEBSITES_ENABLE_APP_SERVICE_STORAGE` setting is unspecified or set to true, the /home/ directory will be shared across scale instances, and files written will persist across restarts. Explicitly setting WEBSITES_ENABLE_APP_SERVICE_STORAGE to false will disable the mount.