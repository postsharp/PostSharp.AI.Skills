---
uid: caching
title: "Caching"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses the use of PostSharp for simplifying caching in application development, reducing complexity, and avoiding errors associated with manual implementation. It also outlines various related topics."
---
# Caching

Caching is a great way to improve the latency of an application. Traditionally, when you implement caching, you need to play with the API of the caching framework (such as <xref:System.Runtime.Caching.MemoryCache>) or caching server (such as Redis) and to include moderately complex logic in your source code to generate the cache key, check the existence of the item in the cache, and add the item into the cache. Another source of complexity stems from removing items from the cache when the source data is updated. Implementing caching manually is time-consuming and error-prone: it is easy to generate inconsistent cache keys between read and update methods. 

PostSharp allows you to dramatically reduce the complexity of caching. It allows you to cache the return value of a method as a function of its arguments with just a custom attribute, namely the <xref:PostSharp.Patterns.Caching.CacheAttribute> aspect. The <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> aspect and the <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation> API offer a strongly-typed way to invalidate cached methods. Additionally, PostSharp is independent from the caching framework or server (called caching *backend*), so you can choose from several backends or implement an adapter for your own backend. 


## In this chapter

| Section | Description |
|---------|-------------|
| <xref:caching-getting-started> | This article shows how to cache the return values of methods. |
| <xref:caching-logging-interaction> | This article shows how to enable logging of the caching aspect to troubleshoot it. |
| <xref:caching-invalidation> | This article shows how to invalidate cached return values of methods declaratively and imperatively. |
| <xref:caching-dependencies> | This article shows how to invalidate cache items automatically using cache dependencies. |
| <xref:caching-keys> | This article shows how to customize the cache keys that identify cached method return values. |
| <xref:caching-backends> | This article shows how to store cached values in various backends. |
| <xref:caching-pubsub> | This article shows how to invalidate all related in-memory caches in a distributed environment. |
| <xref:caching-value-adapters> | This article describes how to cache return values of methods that cannot be cached directly, such as instances of <xref:System.Collections.Generic.IEnumerable`1> or <xref:System.IO.Stream>.  |
| <xref:cache-locking> | This article explains how you can prevent the same method from being executed with the same arguments at the same time by using locking. |

## See Also

**Reference**

<xref:PostSharp.Patterns.Caching.CacheAttribute>
<br><xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute>
<br><xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute>
