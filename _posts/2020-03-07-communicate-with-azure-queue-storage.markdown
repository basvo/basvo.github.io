---
layout: post
title:  "Communicate between applications with Azure Queue Storage"
date:   2020-03-07 10:24:00 +0100
categories: azure queue storage
---

## Communicate between applications with Azure Queue Storage

A storage queue is a high-performance message buffer that can act as a broker between two components.

The use of storage queues is interesting in scenarios that can have spikes in load. At times of high demand, the queue will grow in length, but no messages will be lost. When demand drops to normal levels the application will catch up by working through the queue backlog.

### What is Azure Queue storage

Azure Queue storage is an Azure service that implements cloud-based queues. Each queue maintains a list of messages. A queue can be accessed by using a REST API or a client library. You can have one or more *sender* components, and one or more *receiver* components. Sender components add the messages to the queue. Receiver components retrieve the messages from the front of the queue for processing. This is also called FIFO (First In, First Out).

![An illustration showing a high-level architecture of Azure Queue storage](https://docs.microsoft.com/en-us/learn/modules/communicate-between-apps-with-azure-queue-storage/media/2-queue-overview.png)

After the receiver gets a message, that message remains in the queue but is invisible for 30 seconds. If the receiver crashes or experiences a power failure during processing, then it will never delete the message from the queue. After 30 seconds, the message will reappear in the queue and another instance of the receiver can process it to completion.

Pricing is based on queue size and number of operations. Charges are also incurred for adding and deleting messages. See [Azure Queue storage pricing](https://azure.microsoft.com/pricing/details/storage/queues/) for details.

A single queue can be up to **500 TB** in size. This allows it to store *millions* of messages. The target throughput for a single queue is 2000 messages per second. This means it can handle high-volume scenarios.

Queues are a tool to let your application scale automatically based on demand. There are other services in Azure that scale automatically. For example, the **Autoscale** feature is available on Azure Virtual Machine scale sets, App Service plans and App Service environments. This feature allows you to add capacity by defining rules to identify periods of high demand. While autoscaling responds to demand quickly, it is not instantaneous. By contrast, Azure Queue storage instantaneously handle high demand by storing messages until processing resources are available.

### What is a message

A message in a queue is a byte array of up to 64 KB. If you need a larger payload you can combine queues and blobs – passing the URL to the actual data (stored as a Blob) in the message. This approach would allow you to enqueue up to 200 GB for a single item.

Message contents are not interpreted at all by any Azure component. Your code is responsible for generating and interpreting the custom format. 

### Creating a storage account

A queue must be part of a storage account. You should consider the following settings when creating a storage account that will contain queues:

- Queues are only available as part of Azure general-purpose storage accounts (v1 or v2)
- The **Access tier** setting for StorageV2 accounts does not affect queues. It only applies to Blob storage
- The location should be close to either the source or destination components (preferably both)
- Depending on redundancy needs you should choose **Locally Redundant Storage (LRS)** or **Geo-Redundant Storage (GRS)**. LRS is low-cost but vulnerable for disasters affecting the entire data center. GRS replicates to other Azure data centers but is more costly.
- The performance tier can be **Standard** (magnetic drives) or **Premium** (SSD). Only consider Premium if queue length becomes long and time to access messages should be minimized.
- Require secure transfer when sensitive information passes through the queue. When enabled this ensures all connections to the queue are encrypted with Secure Sockets Layer (SSL).

## Identify a queue

 To access a queue, you need three pieces of information:

1. Storage account name
2. Queue name
3. Authorization token

Every queue has a name that you assign during creation. The name must be unique within your storage account but doesn't need to be globally unique (unlike the storage account name). The combination of your storage account name and your queue name uniquely identifies a queue.

### Access authorization

Every request to a queue must be authorized and there are several options to choose from.

| Authorization Type          | Description                                                  |
| :-------------------------- | :----------------------------------------------------------- |
| **Azure Active Directory**  | Role-based authentication based on AAD credentials.          |
| **Shared Key**              | An encrypted key signature associated with the storage account. It provides *full access* to the storage account. |
| **Shared access signature** | A generated URI that grants limited access to objects in your storage account to clients. Access can be restricted to specific resources, permissions, and date/time range. |

It is recommended to use shared access signature (SAS) or Azure Active Directory (AAD) in production scenarios.

## Accessing queues

You access a queue using a REST API. The URL is formatted as `http://<storage-account>.queue.core.windows.net/<queue-name>` An `Authorization` header must be included with every request. The value can be any of the three authorization styles.

The Azure Storage Client Library for .NET is a library provided by Microsoft that formulates REST requests and parses REST responses. The library provides types to represent each of the objects you need to interact with:

- `CloudStorageAccount` represents your Azure storage account.
- `CloudQueueClient` represents Azure Queue storage.
- `CloudQueue` represents one of your queue instances.
- `CloudQueueMessage` represents a message.

You will use these classes to get programmatic access to your queue. The library has both synchronous and asynchronous methods; you should prefer to use the asynchronous versions to avoid blocking the client app.

This example shows how to connect to a queue:

```c#
CloudStorageAccount account = CloudStorageAccount.Parse(connectionString);
CloudQueueClient client = account.CreateCloudQueueClient();
CloudQueue queue = client.GetQueueReference("myqueue");
```

A common pattern is that the sender application should always be responsible for creating the queue. This keeps your application more self-contained and less dependent on administrative set-up.

The client library exposes a `CreateIfNotExistsAsync` method that will create the queue if it does not exist, or return `false` if it already exists:

```c#
CloudQueue queue;
//...
await queue.CreateIfNotExistsAsync();
```

To send a message, you can use the `CloudQueueMessage` object:

```c#
var message = new CloudQueueMessage("your message here");
CloudQueue queue;
//...
await queue.AddMessageAsync(message);
```

In the receiver, you get the next message, process it, and then delete it after successful processing:

```c#
CloudQueue queue;
//...
CloudQueueMessage message = await queue.GetMessageAsync();
if (message != null)
{
    // Process the message
    //...
    await queue.DeleteMessageAsync(message);
}
```

