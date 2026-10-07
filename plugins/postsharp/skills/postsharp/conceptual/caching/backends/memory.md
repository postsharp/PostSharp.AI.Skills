---
uid: caching-memory
redirect_from:
  - caching/backends/caching-memory
title: "Using In-Memory Cache"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use in-memory caching in PostSharp. It provides instructions for the two backend classes, based on MemoryCache and IMemoryCache."
---
# Using In-Memory Cache

For local, in-memory caching, PostSharp offers two backend classes:

* <xref:PostSharp.Patterns.Caching.Backends.MemoryCachingBackend> relies on <xref:System.Runtime.Caching.MemoryCache>. It is included in the PostSharp.Patterns.Caching package and is available on all supported target frameworks.

* <xref:PostSharp.Patterns.Caching.Backends.MemoryCacheBackend> relies on `Microsoft.Extensions.Caching.Memory.IMemoryCache`. It is included in the PostSharp.Patterns.Caching.IMemoryCache package and requires .NET Standard 2.0 or later.


## In-memory caching with MemoryCache

To use the <xref:System.Runtime.Caching.MemoryCache> class to store cached values in memory, assign an instance of the <xref:PostSharp.Patterns.Caching.Backends.MemoryCachingBackend> class to the <xref:PostSharp.Patterns.Caching.CachingServices.DefaultBackend> property.

```csharp
CachingServices.DefaultBackend = new MemoryCachingBackend();
```

By default, the <xref:System.Runtime.Caching.MemoryCache.Default> instance is used. To use another instance of the <xref:System.Runtime.Caching.MemoryCache> class, pass it to the constructor of the <xref:PostSharp.Patterns.Caching.Backends.MemoryCachingBackend> class.

```csharp
MemoryCache cache = new MemoryCache( "myCache" );
CachingServices.DefaultBackend = new MemoryCachingBackend( cache );
```

See [MSDN](https://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache(v=vs.110).aspx) for details on the <xref:System.Runtime.Caching.MemoryCache> class. 


## In-memory caching with IMemoryCache

To store cached values in an instance that implements the `Microsoft.Extensions.Caching.Memory.IMemoryCache` interface:

1. Add a reference to the [PostSharp.Patterns.Caching.IMemoryCache](https://www.nuget.org/packages/PostSharp.Patterns.Caching.IMemoryCache/) package. 


2. Assign an instance of the <xref:PostSharp.Patterns.Caching.Backends.MemoryCacheBackend> class to the <xref:PostSharp.Patterns.Caching.CachingServices.DefaultBackend> property. Pass the `IMemoryCache` to the constructor. 

    ```csharp
    IMemoryCache cache = new MemoryCache(new MemoryCacheOptions());
    CachingServices.DefaultBackend = new MemoryCacheBackend(cache);
    ```


See [Cache in-memory in ASP.NET Core on MSDN](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-3.1) for more information on the use of `IMemoryCache`.


