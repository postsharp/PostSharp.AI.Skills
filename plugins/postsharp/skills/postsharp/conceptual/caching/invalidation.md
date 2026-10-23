---
uid: caching-invalidation
redirect_from:
  - caching/caching-invalidation
title: "Removing Items From the Cache"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to remove items from the cache in PostSharp, using either direct invalidation, the InvalidateCache aspect, or by invalidating cache items imperatively."
---
# Removing Items From the Cache

When a method updates an entity, it must also remove any cache item that depends on this entity. One way to achieve this goal is to require the update methods to know precisely which cached methods are dependent on the entity that has been modified, and to remove these methods (with proper arguments) from the cache. We call this scenario *direct* cache invalidation. 

The benefit of direct invalidation is that it does not require a lot of resources on the caching backend. However, this approach has a big disadvantage: it exhibits an imperfect separation of concerns. Update methods need to have a precise knowledge of cached methods (typically read methods), so update methods need to be modified whenever a read method is added. For an approach with better abstraction, see <xref:caching-dependencies>. 


## Invalidating cache items using the [InvalidateCache] aspect

You can add the <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> aspect to a method (called the *invalidating method*) to cause any call to this method to remove from the cache the value of one or more other methods. Parameters of both methods are matched by name and type. If any parameter of the cached method cannot be matched with a parameter of the invalidating method, you will get a build error (unless the parameter has the <xref:PostSharp.Patterns.Caching.NotCacheKeyAttribute> custom attribute). The order of parameters is not considered. 

> [!NOTE]
> By default, the <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> aspect looks for the cached method in the current type. You can specify a different type using the alternative constructor of the custom attribute. When you invalidate a non-static method (unless the instance has been excluded from the cache key by setting the <xref:PostSharp.Patterns.Caching.CacheAttribute.IgnoreThisParameter> property to `true`), you can do it only from a non-static method of the same type or of a derived type.
>
> If there are more invalidated methods of the same name for one invalidating method, a build error is emitted. To enable invalidation of all the matching overloads by the one invalidating method, set the <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute.AllowMultipleOverloads> property to `true`. 


### Example

In this example, the `Update` method invalidates the cached return value of the `GetValue` method. 

[!code-csharp[](../../code/samples/Caching/Core/CachingInvalidation.cs#example)]

The output of this sample is:

[!code-text[](../../code/samples/Caching/Core/CachingImperativeInvalidation.txt)]


## Invalidating cache items imperatively

Instead of annotating invalidating methods with a custom attribute, you can call one of the overloads of the <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation.Invalidate``1(System.Func{``0})> method. 


### Example

In this example, the cached return value of the `GetValue` method is invalidated by calling one of the overloads of the <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation.Invalidate``1(System.Func{``0})> method. 

[!code-csharp[](../../code/samples/Caching/Core/CachingImperativeInvalidation.cs#example)]

The output of this sample is:

[!code-text[](../../code/samples/Caching/Core/CachingInvalidation.txt)]

The overloads of <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation> act on <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>. When the class of the cached method receives its caching service from dependency injection, call the <xref:PostSharp.Patterns.Caching.ICachingService.Invalidate(System.Reflection.MethodInfo,System.Object,System.Object[])> method of this service instead, with the method, the instance and the arguments (see <xref:caching-dependency-injection>).


