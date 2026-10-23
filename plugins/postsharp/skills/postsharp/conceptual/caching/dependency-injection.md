---
uid: caching-dependency-injection
title: "Using Caching with Dependency Injection"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how classes with cached methods receive their ICachingService from a dependency injection container such as Microsoft.Extensions.DependencyInjection, with [CacheConfiguration( UseDependencyInjection = true )], and how the caching service, its backend and its profiles are configured in the container."
---
# Using Caching with Dependency Injection

By default, the cached methods use one cache for the whole process: <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>, which you configure at startup (see <xref:caching-getting-started>). In an application whose services are created by a dependency injection container, for instance an ASP.NET Core application, you can instead register the cache in the container. The classes that have cached methods then receive the cache from their constructors, as they receive their other dependencies.

This has the following benefits:

* The cache is configured in the *composition root* of the application, which is the code that registers the services in the container, and it can depend on them, for instance on the `IMemoryCache` of the container.
* Each container has its own cache, which it disposes with the application. Two applications, or two tests, in the same process do not share their cache.
* The cache is an interface, <xref:PostSharp.Patterns.Caching.ICachingService>, which a test can replace.


## The caching service

<xref:PostSharp.Patterns.Caching.ICachingService> represents a cache: the backend that stores the items (<xref:PostSharp.Patterns.Caching.ICachingService.Backend>), the key builder that computes their keys (<xref:PostSharp.Patterns.Caching.ICachingService.KeyBuilder>), the caching profiles (<xref:PostSharp.Patterns.Caching.ICachingService.Profiles>), and the operations that the caching aspects call, including the invalidation methods. <xref:PostSharp.Patterns.Caching.CachingService> is its implementation. Pass the backend, and optionally a key builder, to its constructor.

The methods that do not receive a service from a container use <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>. This property returns the result of the <xref:PostSharp.Patterns.Caching.CachingServices.DefaultServiceLocator> delegate, which returns a <xref:PostSharp.Patterns.Caching.CachingService> created by PostSharp unless you replace it. Setting the delegate to `null` restores the default one. The delegate runs on each call to a cached method, so it must be fast and return the same object every time, for instance a singleton resolved from a container. Set it before you configure <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>, because the configuration does not move to the service that a new delegate returns.


## Receiving the caching service from the container

To make the classes with cached methods receive their caching service from the container:

1. Add the <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> custom attribute to the assembly, to the classes or to a base class, and set its <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.UseDependencyInjection> property to `true`.

2. Register an <xref:PostSharp.Patterns.Caching.ICachingService> in the container, typically a singleton <xref:PostSharp.Patterns.Caching.CachingService> created with the backend of your choice (see <xref:caching-backends>).

3. Let the container create the objects of these classes.

The setting applies to every class that has a method with <xref:PostSharp.Patterns.Caching.CacheAttribute> or <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute>. PostSharp reads it at build time from the declaring class of the method, then from its base classes, then from the assembly.

At build time, PostSharp appends a parameter of type <xref:PostSharp.Patterns.Caching.ICachingService> to the instance constructors of these classes, and of the classes that derive from them, in the same project or in another project that PostSharp processes. A constructor that already has a parameter of type <xref:PostSharp.Patterns.Caching.ICachingService> keeps its signature, and the aspects use this parameter. For the other constructors, PostSharp creates a new constructor with the additional parameter, and the original constructor becomes a forwarder to it. A `public` forwarder becomes `protected internal`. For the details of this transformation, see <xref:constructor-dependencies>.

> [!IMPORTANT]
> <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.UseDependencyInjection> changes the public constructors of the classes. When such a class is public and used by other assemblies, this is a binary breaking change for them.


## Example

The following `ProductCatalog` class has a cached method and an invalidating method. Its source code declares only the repository in its constructor:

[!code-csharp[](../../code/samples/Caching/DependencyInjection/CachingDependencyInjection.cs#catalog)]

The catalog is a scoped service, so each request has its own catalog. <xref:PostSharp.Patterns.Caching.CacheAttribute.IgnoreThisParameter> excludes the instance from the cache key, so that all the catalogs share the cached items. The following controller uses the catalog, and knows nothing about caching:

[!code-csharp[](../../code/samples/Caching/DependencyInjection/CachingDependencyInjection.cs#controller)]

The composition root registers the `IMemoryCache` of Microsoft.Extensions.Caching.Memory, a <xref:PostSharp.Patterns.Caching.Backends.MemoryCacheBackend> over it (see <xref:caching-memory>), and the <xref:PostSharp.Patterns.Caching.CachingService>, whose default profile it configures:

[!code-csharp[](../../code/samples/Caching/DependencyInjection/CachingDependencyInjection.cs#registration)]

The container passes the repository and the caching service to the constructor of the catalog. The following code handles three requests, each in its own scope, as ASP.NET Core does for each HTTP request:

[!code-csharp[](../../code/samples/Caching/DependencyInjection/CachingDependencyInjection.cs#requests)]

The second request reads the product from the cache. The third one renames the product, which invalidates the cached item:

[!code-text[](../../code/samples/Caching/DependencyInjection/CachingDependencyInjection.txt)]

Because the backend is a singleton of the container, the container disposes it when it is disposed.


## Objects created with `new`

The objects of these classes are normally created by the container, by `ActivatorUtilities`, or by the constructor of a derived class. When the source code creates an object with `new`, it calls a forwarder, which passes `null` for the caching service. The constructor then gets the service from <xref:PostSharp.Extensibility.RunTimeServiceLocator> (see [Objects created without a container](xref:constructor-dependencies#objects-created-without-a-container)).

PostSharp Caching registers <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService> in <xref:PostSharp.Extensibility.RunTimeServiceLocator> as the <xref:PostSharp.Patterns.Caching.ICachingService>, unless the application registered its own service first. An object created with `new` therefore uses the default service, as a static method does. To give it another service, register it at startup:

<!-- not-compiled: needs the ServiceProvider of an application -->
```csharp
RunTimeServiceLocator.RegisterService<ICachingService>( () => application.GetRequiredService<ICachingService>() );
```


## Static methods and structs

A static method, or a method of a struct, has no object that receives the service from the container. It uses <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>, and PostSharp reports the message <xref:cac011?text=CAC011>. Its severity is the value of the <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.StaticMethodFallbackSeverity> property of <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute>:

* <xref:PostSharp.Extensibility.SeverityType.Warning?text=SeverityType.Warning> by default.
* <xref:PostSharp.Extensibility.SeverityType.Error?text=SeverityType.Error> to require that every cached method receives its service from the container.
* <xref:PostSharp.Extensibility.SeverityType.None?text=SeverityType.None> to accept the fallback silently.

When the method of a struct changes the struct, the change reaches the caller, with or without dependency injection.


## Invalidating cached methods

<xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> invalidates the cache of the object that it receives, so an invalidating method and the methods it invalidates must use the same service. <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> reads its own <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.UseDependencyInjection> setting, and PostSharp reports the error <xref:cac012?text=CAC012> when it invalidates a method whose setting is different.

To invalidate imperatively, call the <xref:PostSharp.Patterns.Caching.ICachingService.Invalidate*> methods of the service: with a <xref:System.Reflection.MethodInfo>, the instance and the arguments to remove one method call result, or with a dependency (see <xref:caching-dependencies>). The overloads of <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation> act on <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>, so they do not invalidate the items of a service received from the container. A class whose constructor declares a parameter of type <xref:PostSharp.Patterns.Caching.ICachingService> receives the same service as its cached methods, and can store it in a field.

The aspects import the caching service of the target class by its type:

* With <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.UseDependencyInjection>, the aspects import a field of exactly the type <xref:PostSharp.Patterns.Caching.ICachingService> that the class already has, or, when there is no such field, a readable instance property of exactly this type. The class then receives no new constructor parameter, and it must initialize this member itself (see [The field](xref:constructor-dependencies#the-field)).
* Without <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.UseDependencyInjection>, a field or a property whose type can be assigned to <xref:PostSharp.Patterns.Caching.ICachingService>, for instance <xref:PostSharp.Patterns.Caching.CachingService>, is used instead of <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>. Two such members cause an error.


## Profiles, formatters and the caching context

Each caching service has its own caching profiles, in its <xref:PostSharp.Patterns.Caching.ICachingService.Profiles> property. <xref:PostSharp.Patterns.Caching.CachingServices.Profiles> configures only the profiles of <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>, so it does not configure the methods of the classes that receive a service from the container. Configure their profiles on the service that you register, as in the example above.

The following settings are shared by all the services of the process:

* The formatters of <xref:PostSharp.Patterns.Caching.CachingServices.Formatters>, which build the cache keys (see <xref:caching-keys>).
* The caching context of <xref:PostSharp.Patterns.Caching.CachingServices.CurrentContext>, and <xref:PostSharp.Patterns.Caching.CachingServices.SuspendDependencyPropagation*>.

When a method cached by one service calls a method cached by another service, the inner method is not added as a dependency of the outer method (see <xref:caching-dependencies>), because its invalidation would not reach the backend of the outer method.


## See Also

**Reference**

<xref:PostSharp.Patterns.Caching.ICachingService>
<br><xref:PostSharp.Patterns.Caching.CachingService>
<br><xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>
<br><xref:PostSharp.Patterns.Caching.CachingServices.DefaultServiceLocator>
<br><xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.UseDependencyInjection>
<br><xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.StaticMethodFallbackSeverity>
<br><xref:PostSharp.Extensibility.RunTimeServiceLocator>

**Other Resources**

<xref:caching-getting-started>
<br><xref:caching-backends>
<br><xref:constructor-dependencies>
<br>[Dependency injection in .NET](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
