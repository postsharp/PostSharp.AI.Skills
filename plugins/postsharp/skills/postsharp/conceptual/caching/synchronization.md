---
uid: caching-pubsub
redirect_from:
  - caching/caching-pubsub
title: "Synchronizing Local In-Memory Caches for Multiple Servers"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to synchronize local in-memory caches for multiple servers using PostSharp. It offers solutions for distributed cache invalidation using Azure Service Bus pub/sub and Redis pub/sub."
---
# Synchronizing Local In-Memory Caches for Multiple Servers

Caching in distributed applications can be a difficult problem. When there are several instances of an application running simultaneously (typically web sites or web services deployed into the cloud or web farms), you have to make sure that the cache is properly invalidated for all instances of the application.

A typical answer to this issue is to use a centralized cache server (or a cluster of cache servers) that solves this problem for you. For instance, you can use a Redis server or a Redis cluster. However, running a cache server, and even more so a cache cluster, has a cost, and it does not always pay off for medium applications such as the website of a small business.

An alternative solution to the problem of distributed caching is to have a local in-memory cache in each instance of the application. Instead of using a shared distributed cache, each application instance caches its own data into its own local cache. However, when one instance of the application modifies a piece of data, it needs to make sure that all instances remove the relevant items from their local caches. This is called *distributed cache invalidation*. It can be achieved easily and cheaply with a publish/subscribe (Pub/Sub) message bus such as Azure Service Bus, much less expensive than a cache cluster. 

PostSharp allows you to easily add pub/sub cache invalidation to your existing PostSharp caching.

The principal inconvenience of pub/sub invalidation is that there is some latency in the invalidation mechanism, that is, different instances of the application can see different data for a few dozen milliseconds.


## Using Azure Service Bus pub/sub for distributed invalidation

> [!WARNING]
> `AzureCacheInvalidator` and `AzureCacheInvalidator2` are deprecated since PostSharp 2024.0. They are replaced by <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidator>.

### Configuring Azure Service Bus invalidation

1. Add in-memory local caching to your application as described in <xref:caching-memory>. 


2. Add a reference to the [PostSharp.Patterns.Caching.Azure](https://www.nuget.org/packages/PostSharp.Patterns.Caching.Azure/) NuGet package. 


3. Go to the Microsoft Azure portal, open the **Service Bus** panel and create a new **Topic**. Choose a small value for the time-to-live setting, for instance 30 seconds. See the [Microsoft Azure website](https://azure.microsoft.com/en-us/services/service-bus/) for details. 


4. In the Microsoft Azure portal, create a **Shared access policy**. Include the **Send** and **Listen** rights, and the **Manage** right if you want the invalidator to create topic subscriptions automatically (see below). This policy will be used by your application. 


5. Go to the properties of the newly created policy and copy the primary or secondary connection string to the clipboard.


6. Go back to your source code and find the place where the <xref:PostSharp.Patterns.Caching.Backends.MemoryCachingBackend> is initialized. 

7. Create an instance of the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidatorOptions> class, passing the connection string of the shared access policy you just created and the topic name. You can omit the topic name if the connection string contains an `EntityPath` component. To use a topic subscription that you created yourself, set the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidatorOptions.SubscriptionName> property. See below. 


8. Create an instance of the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidator> class using the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidator.Create(PostSharp.Patterns.Caching.Implementation.CachingBackend,PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidatorOptions)> factory method, passing the existing instance of the <xref:PostSharp.Patterns.Caching.Backends.MemoryCachingBackend> and the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidatorOptions>. Assign the new <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidator> to the <xref:PostSharp.Patterns.Caching.ICachingService.Backend?text=CachingServices.DefaultService.Backend> property. 

### Topic creation strategies

A topic subscription can connect only one client to the Azure server at a time. For example, if you start your application on two different computers but use the same Service Bus topic subscription for both, cache invalidation will not work. It is necessary to have two topic subscriptions and assign one to each application instance.

There are two ways to create topic subscriptions: automatic and manual.

* **Automatic topic management** means that PostSharp will automatically create a topic subscription for each instance of the cache invalidator. This topic subscription will have an auto-generated name and will be deleted automatically when it has been idle for the duration of the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidatorOptions.AutoDeleteOnIdle> property, 5 minutes by default. 
    This strategy requires the shared access policy to include the **Manage** right.

* **Manual topic management** means that you have to create the topic subscriptions yourself using the Azure portal or another management API. In this case, you need to set the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidatorOptions.SubscriptionName> property, but your shared access policy does not need the **Manage** right. 

You select the strategy with the <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidatorOptions.SubscriptionName> property: automatic when it is `null` (the default), manual otherwise.


## Using Redis pub/sub for distributed invalidation

If you are already using Redis for PostSharp caching, it is useless to add another layer of invalidation because this is already taken care of by the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> class. However, if you already have a Redis cluster but you do not want to use it for caching, you can still use it for cache invalidation. An example situation is when the latency of your Redis server is too high for caching but sufficient for cache invalidation. 


### Configuring Redis Pub/Sub invalidation

1. Add in-memory local caching to your application as described in <xref:caching-memory>. 


2. Create an instance of [StackExchange.Redis.ConnectionMultiplexer](https://stackexchange.github.io/StackExchange.Redis/Configuration). See [Redis Pub/Sub documentation](https://redis.io/topics/pubsub/) for details. 


3. Create and configure an instance of the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheInvalidatorOptions> class. 


4. Create an instance of the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheInvalidator> class using the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheInvalidator.Create(PostSharp.Patterns.Caching.Implementation.CachingBackend,StackExchange.Redis.IConnectionMultiplexer,PostSharp.Patterns.Caching.Backends.Redis.RedisCacheInvalidatorOptions)> factory method, passing your instance of <xref:PostSharp.Patterns.Caching.Backends.MemoryCachingBackend> and <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheInvalidatorOptions>. Assign the instance to the <xref:PostSharp.Patterns.Caching.ICachingService.Backend?text=CachingServices.DefaultService.Backend> property. 



### Example

This example shows how to initialize an in-memory caching backend to let it invalidate and be invalidated using Redis Pub/Sub.

[!code-csharp[](../../code/samples/Caching/Backends/CachingPubSub.cs#redis)]


