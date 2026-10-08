---
uid: caching-redis
redirect_from:
  - caching/backends/caching-redis
title: "Using Redis Cache"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions on how to use Redis Cache with PostSharp, including server configuration, setting up PostSharp for Redis caching, adding local in-memory cache, and using dependencies with the Redis caching backend."
---

# Using Redis Cache

Redis is a high-performance, in-memory key-value store widely used for caching scenarios to improve application speed and scalability. This page explains how to integrate Redis as a caching backend in your applications. You’ll learn how to configure your Redis server and your application for optimal performance with PostSharp’s caching features.

Our implementation uses the [StackExchange.Redis library](https://stackexchange.github.io/StackExchange.Redis/) internally. It is compatible with on-premises Redis Cache instances as well as with the [Azure Redis Cache](https://azure.microsoft.com/en-us/services/cache/) cloud service.

## Prerequisites

In theory, this component should work with any Redis implementation supported by `StackExchange.Redis`.

However, we only tested this component with the latest stable Redis version available at the time of release. Older versions or alternative implementations are not officially supported except on an ad-hoc basis for customers with an enterprise support plan.

We performed tests with single-node, primary/replica, and sharded cluster deployments.


## Configuring the Redis server

To prepare your Redis server for use with PostSharp caching:

1. Set up the eviction policy to `volatile-lru` or `volatile-random`. See <https://redis.io/topics/lru-cache#eviction-policies> for details.

    > [!CAUTION]
    > Other eviction policies than `volatile-lru` or `volatile-random` are not supported.


2. If you intend to enable support for dependencies, configure keyspace notifications to `Exe` where:
    - `E` = Keyevent notifications,
    - `x` = Expired events,
    - `e` = Evicted events.

    See <https://redis.io/docs/latest/develop/interact/notifications/> for details.


### Example

Here is an excerpt of your `redis.conf` configuration file:

```text
maxmemory-policy volatile-lru
notify-keyspace-events "Exe"
```

## Configuring the caching backend in PostSharp

To set up PostSharp to use Redis for caching:

1. Add a reference to the [PostSharp.Patterns.Caching.Redis](https://www.nuget.org/packages/PostSharp.Patterns.Caching.Redis/) package.

2. Create an instance of [StackExchange.Redis.ConnectionMultiplexer](https://stackexchange.github.io/StackExchange.Redis/Configuration).

3. Create an instance of the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> class using the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend.Create*> factory method and assign the instance to <xref:PostSharp.Patterns.Caching.CachingServices.DefaultBackend>, passing a <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration> object.

4. Configure logging. <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> uses PostSharp Logging. See <xref:backends> to learn how to plug caching into your logging framework. While tuning performance, we recommend monitoring warnings.

> [!IMPORTANT]
> If one node enables the local cache, all nodes must enable events.

In-memory cache must be enabled globally for the whole backend. It is not possible to enable it at the level of individual classes or methods.

> [!IMPORTANT]
> The caching backend must be set before any cached method is called for the first time.

### Example

```csharp
// Enable logging
LoggingServices.DefaultBackend = new ConsoleLoggingBackend();
LoggingServices.DefaultBackend.DefaultVerbosity.SetMinimalLevel( LogLevel.Warning, LoggingRoles.Caching );

// Connect to Redis
string connectionConfiguration = "localhost";
ConnectionMultiplexer connection = ConnectionMultiplexer.Connect( connectionConfiguration );

// Set Redis as the default backend.
RedisCachingBackendConfiguration redisCachingConfiguration =
    new RedisCachingBackendConfiguration() { KeyPrefix = "MyApp-1.0.1" };
CachingServices.DefaultBackend = RedisCachingBackend.Create( connection, redisCachingConfiguration );
```

### Important configuration settings

- The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix> property should be set to a value that uniquely identifies the version of the data model. This will allow you to run several versions of your application side by side with the same database.
- The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.DefaultExpiration> property is the expiration for items that do not have an explicit expiration and are not marked unremovable. It defaults to 30 days.
- The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.Database> property allows you to select the logical database. It defaults to -1, which selects the default database of the connection (database 0 unless the connection configuration specifies another one). If you run several applications on the same standalone Redis node, it’s recommended to place each application in a separate database so that you can clear the cache easily with the `FLUSHDB` Redis command. Note that in Redis Cluster mode only database 0 is available, so you must differentiate applications using the `KeyPrefix` instead.

## Adding a local in-memory cache to a remote Redis server

For higher performance, you can add an additional, in-process layer of caching between your application and the remote Redis server. To enable the local cache, set the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.IsLocallyCached> property to `true`.

The benefit of using local caching is to reduce latency between the application and the Redis server and to decrease CPU load due to object deserialization.

This feature relies on Redis' Pub/Sub subsystem to propagate cache invalidation events. This synchronization is asynchronous, which means that a key invalidation on a node does not wait until the key is invalidated on all clients. Therefore, different nodes may have different views of the cache — even if you are using a single Redis node. Since Redis' primary/replica synchronization is also asynchronous, having an in-memory caching layer does not fundamentally differ from a primary/replica setup. In this case, however, processing delays may be caused by the application nodes if they are overloaded, even if the Redis cluster is synchronizing fast.

The following code enables the in-memory cache:

```csharp
RedisCachingBackendConfiguration redisCachingConfiguration =
     new RedisCachingBackendConfiguration() { IsLocallyCached = true, SupportsEvents = true };
CachingServices.DefaultBackend = RedisCachingBackend.Create( connection, redisCachingConfiguration );
```

> [!IMPORTANT]
> If one node enables the local cache, all nodes must enable events.

In-memory cache must be enabled globally for the whole backend. It is not possible to enable it at the level of individual classes or methods.

## Using dependencies with the Redis caching backend

Support for dependencies is disabled by default with the Redis caching backend because it has an important performance and deployment impact:
- Performance impact: the dependency graphs need to be stored and maintained in Redis.
- Deployment impact: the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector> component, which cleans up dependencies when cache items expire or are evicted, needs to run continuously.

To use dependencies with the Redis caching backend:

1. Set the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.SupportsDependencies?text=RedisCachingBackendConfiguration.SupportsDependencies> property to `true`.

2. Make sure that at least one instance of the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector> class is alive at any moment (whether the application is running or not). It must be configured with the same <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix> and <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.Database> as your main application. You may package the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector> into a separate application or cloud service.

    This component performs two tasks:

    - It subscribes to `__keyevent@<db>__:expired` and `__keyevent@<db>__:evicted` notifications, where `<db>` is the database index, and cleans up the dependency graphs in real time. This process generally works well, but it is temporarily disabled when the system is overloaded.
    - It periodically scans the entire database and removes any unreachable data (garbage).

    If you choose to enable dependencies with Redis, you need to ensure that at least one instance of the cache GC process is running. It is acceptable to have several instances running, but since all instances will compete to process the same messages, it’s better to have a single instance running.

The dependencies feature of <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> is designed so that the cache is observationally consistent from the client’s point of view during normal operation. Read operations are fast and transactionless. Write operations are optimized for performance and read consistency, but may leave garbage behind in case of a race condition (later cleaned by the GC process).

Garbage may therefore be due to three factors:

- The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector> component was not running when a cache eviction or expiration happened.
- The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector> component was temporarily disabled because of high system load.
- There was a race condition in setting a cache value (the version that loses becomes garbage).

### Limitations

Direct forward dependencies (i.e. the dependencies that a cache item directly depends on, at the first level) must be kept reasonably low because they are all loaded into memory as an array at the same time. The system can probably handle hundreds or thousands of forward dependencies, but not millions.

The number of items depending on a single dependency is only limited by your Redis instance. They are processed in a cursor-based manner and are never all simultaneously loaded in memory.

## Resilience

As with any part of a distributed system, <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> is a complex component that must be tuned and monitored with care.

### Exception handling

Failures are highly likely if your system is overloaded. Here is how they are handled:

1. Foreground operations fail fast. In case of a cache exception, we skip the cache. Additionally, when a writing operation fails, we schedule a background recovery task to invalidate the cache item, based on the principle that the worst outcome is an inconsistent cache.
2. Background operations are retried according to a retry policy (see <xref:PostSharp.Patterns.Caching.Resilience.IRetryPolicy>) that can be configured through <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.TransactionRetryPolicy> and <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.BackgroundRecoveryRetryPolicy> properties. The default retry policy implements an exponential backoff delay strategy.

The exception handling policy described above is abstracted by the <xref:PostSharp.Patterns.Caching.Resilience.IExceptionHandlingPolicy> interface and configured by the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.ExceptionHandlingPolicy> property.

### Performance tuning

With any level of load, it is recommended to enable cache key compression by assigning the <xref:PostSharp.Patterns.Caching.CachingServices.DefaultKeyBuilder?text=CachingServices.DefaultKeyBuilder> property to a <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder> instance with a non-default <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyHashingAlgorithm>.

If you expect high load, it is recommended to tune the following <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration> properties:

- <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.BackgroundTasksMaxConcurrency> is critical for the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector> component. It should be high enough to utilize the maximum resources of your deployment, but small enough not to allow for a large operation backlog to form.
- <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.BackgroundTasksOverloadedThreshold> is used only by the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector>. It will cause the component to stop processing eviction and expiration notifications if the operation backlog is too large. In this case, GC would rely on periodic database scanning.

The following <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollectorOptions> properties must also be properly tuned:
- The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollectorOptions.CacheCleanupDelay> property defines the delay between the component's initialization and the first cleanup, and then between subsequent cache cleanups; it defaults to 4 hours. Cleaning up the database too frequently is an unnecessary performance overhead, but doing it too late degrades performance even more. If the database contains too much garbage, Redis will start evicting useful data, affecting your application's performance. However, it will never evict garbage. That is why you should increase the cache cleanup frequency if you see high memory usage or high levels of evictions.
- The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollectorOptions.CacheCleanupOptions> property affects the cleanup process. It is important to keep the cleanup slow enough to avoid impacting your application's performance, but fast enough to finish before Redis runs out of memory. The <xref:PostSharp.Patterns.Caching.Implementation.CacheCleanupOptions.WaitDelay> is an artificial delay between processing each cache key, defaulting to 100 ms.

Note that you can run a manual cleanup by calling the <xref:PostSharp.Patterns.Caching.Implementation.CachingBackend.CleanUpAsync*?text=CachingBackend.CleanUpAsync> method. Do not run this method with the default options on a production database; these options are optimized for cleanup performance and may overload your server.

### Monitoring

The following metrics are relevant for assessing the health of your caching setup:

- Number of cache evictions per second. A high number might indicate either insufficient caching memory or an ineffective caching strategy—caching things that are not worth it (too many cache misses).
- Number of cache expirations per second. A high number might indicate too-short expiration delays.
- Number of warnings per minute in the caching component. A high number indicates that your system is overloaded.

If you want to gather statistics about cache hits and misses, you can do so by implementing a <xref:PostSharp.Patterns.Caching.Implementation.CachingBackendEnhancer> that overrides the <xref:PostSharp.Patterns.Caching.Implementation.CachingBackend.GetItemCore*> and <xref:PostSharp.Patterns.Caching.Implementation.CachingBackend.GetItemAsyncCore*> methods (a `null` return value means a cache miss).

## Data schema and complexity analysis

### Data schema

When dependencies are enabled, <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> relies on these keys:

1. Item version: `<prefix>:<schema-version>:{<item-key>}:ver` — unique string identifier.
2. Item value: `<prefix>:<schema-version>:{<item-key>}:val:<item-version>` — a list with values: `[<item-value>, <item-sliding-expiration>, <tag0>, <tag1>, ... <tagn>]`.
3. Forward dependencies: `<prefix>:<schema-version>:{<item-key>}:fdep:<item-version>` — list of `<dependency-key>`.
4. Backward dependencies: `<prefix>:<schema-version>:{<dependency-key>}:bdep` — hash set of `<item-version>:<item-key>`.

When dependencies are disabled, only the item value record is used.

In this description, elements between angle brackets are placeholders and mean the following:

- `<item-key>` is the cache key (as generated by <xref:PostSharp.Patterns.Caching.Implementation.CacheKeyBuilder>), where `{`, `}` and `:` have been escaped.
- `<item-value>` is the serialized cache value.
- `<prefix>` is the value of the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix> property.
- `<schema-version>` is the version of the data schema internal to <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend>.
- `<item-version>` is a randomly generated item version.
- `<dependency-key>` is either a dependency key or a cache item key, when the cache item is itself a dependency (recursive dependencies), where `{`, `}` and `:` have been escaped.

Note the use of `{..}` brackets around `<item-key>`, marking `<item-key>` as a hashtag. It guarantees that all Redis keys related to the same logical cache item are stored on the same cluster slot and can be processed atomically in a transaction.

### Big O analysis

This analysis uses the following parameters:

- `Items` is the number of items.
- `Dependencies` is the number of dependencies.
- `KeySize` is the average size of item or dependency keys (after compression, if enabled).
- `ValueSize` is the average size of item values (i.e. the serialized data).
- `Dependencies_Per_Item` is the average number of dependencies per item (first level, not recursive).
- `Items_Per_Dependency` is the average number of items that a dependency or another item depends on (first level, not recursive).
- `Recursive_Items_Per_Dependency` is the average number of items that a dependency or another item depends on, recursively.

Memory usage is the sum of:

- Version: `O(Items * KeySize)`
- Value: `O(Items * (KeySize + ValueSize))`
- Forward dependencies: `O(Items * KeySize * (1 + Dependencies_Per_Item))`
- Backward dependencies: `O((Dependencies + Items) * KeySize * Items_Per_Dependency)`

Total memory usage: `O( Items * ValueSize + Items * KeySize * ( 2 + Dependencies_Per_Item + Items_Per_Dependency) + Dependencies * KeySize * Items_Per_Dependency)`

Time complexity of operations is the following:

- Get: `O(1)`
- Set: `O(Dependencies_Per_Item * (1 + Recursive_Items_Per_Dependency))`
- Invalidate: `O(Recursive_Items_Per_Dependency * Dependencies_Per_Item)`
- Clean up: `O(Items * Dependencies_Per_Item + Dependencies * Items_Per_Dependency)`

Race conditions affecting performance can occur when several operations attempt to set the same key. In this case, Redis transactions are used to achieve consistency, and they might be repeated in case of a race condition. Adding items that share the same dependencies does not cause race conditions and does not affect performance.

## Clearing the cache

To remove all cache keys, you can:

- Use the `FLUSHDB` Redis command to delete all keys in the selected database, even those that don’t match the prefix.
- Use the `SCAN <prefix>:*` command to identify all cache keys, then use `UNLINK` (preferred) or `DEL` for each key.
- Use the <xref:PostSharp.Patterns.Caching.Implementation.CachingBackend.ClearAsync*> method, which performs a `SCAN <prefix>:<schema-version>:*` then `UNLINK` for each key.

## Updating the data schema

When you update your application with a new data schema, it is important to think carefully about the deployment process.

There are two circumstances in which the data schema can change:

* When your application has a new and compatible serialization schema for cached data, you must set the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix> property to a different value.
* When the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> component is updated to a new schema, we will update the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend.SchemaVersion?text=RedisCachingBackend.SchemaVersion> constant.

The <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix> and <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend.SchemaVersion?text=RedisCachingBackend.SchemaVersion> properties are prepended to Redis keys. They ensure that the new version of your application can co-exist with the old version without conflicts.

However, after an update, the new <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCacheDependencyGarbageCollector> process will not clean up the data of the old version. You must do that manually after decommissioning the old version.

There are two ways to do it:

- Easily: by purging the whole database (`FLUSHDB`).
- Selectively: by removing all keys that do not match the `<prefix>:<schema-version>:*` pattern. Starting from PostSharp 2025.1.8, you can use the <xref:PostSharp.Patterns.Caching.Implementation.CachingBackend.ClearAsync*> method, but remember to do that with the old application version.

## Troubleshooting

### Observing the cache

- Make sure that `LoggingServices.DefaultBackend` has been properly configured so that the logging messages generated by <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> are routed to a service where you can monitor them.

- If necessary, increase the logging verbosity to `Info` (suitable for production in troubleshooting situations) or `Debug` (extremely verbose and never suitable for production). If you don't see any message in `Debug` verbosity, it means that logging is not properly configured.

    ```csharp
    LoggingServices.DefaultBackend.DefaultVerbosity.SetMinimalLevel( LogLevel.Info, LoggingRoles.Caching );
    ```

- Use the Redis `MONITOR` command in Redis CLI, or Redis Insight's profiler tool.

- Use the Redis pattern subscription to keyevent channels, e.g. `PSUBSCRIBE __keyevent@0__*` in Redis CLI, or use Redis Insight's Pub/Sub tool and subscribe to the pattern `__keyevent@0__*`.

### Common issues

#### Problem: Cache accesses never hit, always miss

**Cause**: <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackend> cannot connect to Redis and the `AbortOnConnectFail` option of `StackExchange.Redis` is set to `false`. You should see errors in your logs. If you don't, logging is not properly configured.

**Remedy**: Check that you can connect to the Redis server, for instance using `connection.GetDatabase().Ping()`.

#### Problem: The collector component reports dozens of errors per minute

**Cause**: The collector is overloaded because of excessive evictions and expirations.

Remedies:

- If expirations are excessive, increase the cache item expiry delay.
- If evictions are excessive, remove caching from methods with a high miss ratio.
- If the problem is intermittent, tune the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.BackgroundTasksMaxConcurrency> and <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.BackgroundTasksOverloadedThreshold>.

#### Problem: InvalidCacheItemException or InvalidCastException are logged after an application upgrade.

**Cause**: The serialization of new and old data classes is not compatible with each other, but the two versions of the application use the same <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix> and <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.Database> properties.

**Remedies**:

- Use a different value for the <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.KeyPrefix> or <xref:PostSharp.Patterns.Caching.Backends.Redis.RedisCachingBackendConfiguration.Database> property when you update cached classes.
- Use a serializer that makes the data contract explicit, i.e. <xref:PostSharp.Patterns.Caching.Serializers.DataContractSerializer> or <xref:PostSharp.Patterns.Caching.Serializers.JsonCachingSerializer> instead of <xref:PostSharp.Patterns.Caching.Serializers.BinarySerializer> or <xref:PostSharp.Patterns.Caching.Serializers.PortableSerializer> — and maintain serialization compatibility when you update the classes.

## See Also

**Other Resources**

[StackExchange.Redis documentation](https://stackexchange.github.io/StackExchange.Redis/)
<br>[Redis Insight](https://redis.io/insight/) for monitoring during development
