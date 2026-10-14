---
uid: caching-getting-started
redirect_from:
  - caching/caching-getting-started
title: "Caching Method Return Values"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on how to cache method return values using PostSharp's caching method. It includes configuring cache behavior, using custom attributes for caching, and disabling caching at run time."
---
# Caching Method Return Values

Suppose you have a time-consuming method that always returns the same return value when called with the same arguments. You decide to cache it. This topic describes how to proceed.


## Caching the return value of a method


### Adding caching to a method

1. Add a reference to the [PostSharp.Patterns.Caching](https://www.nuget.org/packages/PostSharp.Patterns.Caching/) package. 


2. Add the <xref:PostSharp.Patterns.Caching.CacheAttribute> custom attribute to the method that you want to cache. Such a method is called the *cached method*. 


3. Now you need to specify which caching framework or caching server is to be used by the <xref:PostSharp.Patterns.Caching.CacheAttribute> aspect. We call this the *caching backend*. You must specify the caching backend by setting the <xref:PostSharp.Patterns.Caching.CachingServices.DefaultBackend> property. See <xref:caching-backends> for a list of caching backends. 

    > [!IMPORTANT]
    > The caching backend has to be set before any cached method is called for the first time.


4. Unless all method parameters are intrinsic types such as `int` or `string`, you need to ensure that the parameter types generate a meaningful cache key. See <xref:caching-keys> for details. 



### Example

In this example, the return value of the `GetNumber` method is cached. 

[!code-csharp[](../../code/samples/Caching/CachingGettingStarted.cs#example)]

The output of this sample is:

[!code-text[](../../code/samples/Caching/CachingGettingStarted.txt)]


## Configuring the cache behavior

The following elements of the <xref:PostSharp.Patterns.Caching.CacheAttribute> aspect behavior can be configured: 

* expiration (absolute and sliding),

* priority,

* auto-reload, and

* enabled/disabled.


## Configuring caching with custom attributes

You can configure the <xref:PostSharp.Patterns.Caching.CacheAttribute> aspect by setting the properties of the <xref:PostSharp.Patterns.Caching.CacheAttribute> custom attribute. The inconvenience of this approach is that you have to repeat the configuration for each cached method. To configure several methods in a single line of code, you can add the <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> custom attribute to the declaring type, a parent of the declaring type, or the declaring assembly. 

When PostSharp processes the <xref:PostSharp.Patterns.Caching.CacheAttribute> aspect on a given method, it looks for configuration in the following order: 

* the <xref:PostSharp.Patterns.Caching.CacheAttribute> itself, 

* a <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> attribute on the declaring class of the cached method, 

* a <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> attribute on any parent class of the cached method (starting from the declaring class to the parent class), 

* a <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> added to the assembly declaring the cached method. Note that all <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> assembly-level attributes defined in a different assembly than the current one are ignored. 

* the caching profile (see below).

Configuration is defined on a per-property basis. For each property of the cache aspect, the value set by the highest item in the preceding list wins.


### Example

In the following example, the absolute expiration of cache items is set to 60 minutes for methods of the `AccountServices` class, but to 20 minutes for the `GetAccountsOfCustomer` method. 

[!code-csharp[](../../code/samples/Caching/CachingConfiguration.cs#example)]


## Configuring caching with caching profiles

Using custom attributes to configure caching has two major inconveniences: it is hard to share caching configuration between several classes that don't derive from the same parent, and the configuration cannot be modified at run time. To work around these limitations, you can use caching profiles.

Caching profiles are useful in the following scenarios:

* to centralize the configuration of several cached methods (which may belong to different type hierarchies) into a single location;

* to modify the configuration of cached methods (such as expiration settings) at run time; and

* to completely disable or re-enable caching at run time.

Caching profiles are represented by the <xref:PostSharp.Patterns.Caching.CachingProfile> class. They are exposed on the <xref:PostSharp.Patterns.Caching.CachingServices.Profiles> property, which is a collection indexed by the profile name. The default caching profile is accessible via the <xref:PostSharp.Patterns.Caching.CachingServices.CachingProfileRegistry.Default> property. 

> [!NOTE]
> Configuration specified by the <xref:PostSharp.Patterns.Caching.CacheAttribute> aspect and the <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> custom attribute has priority over the configuration of the <xref:PostSharp.Patterns.Caching.CachingProfile>. 


### Using a caching profile

1. Set the <xref:PostSharp.Patterns.Caching.CacheAttribute.ProfileName> property of the <xref:PostSharp.Patterns.Caching.CacheAttribute> aspect or the <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute> custom attribute. 


2. Configure the <xref:PostSharp.Patterns.Caching.CachingProfile> object exposed on the <xref:PostSharp.Patterns.Caching.CachingServices.Profiles> collection. 



### Example

The following code snippet sets the profile name to `Account` for all methods of the `AccountServices` class. 

[!code-csharp[](../../code/samples/Caching/CachingProfiles.cs#services)]

The following code snippet sets the absolute expiration to 60 seconds for all methods using the `Account` profile. 

[!code-csharp[](../../code/samples/Caching/CachingProfiles.cs#configure)]


## Disabling caching at run time

You can disable caching at run time by setting the <xref:PostSharp.Patterns.Caching.CachingProfile.IsEnabled> property of a caching profile to `false`, for instance: 

[!code-csharp[](../../code/samples/Caching/CachingProfiles.cs#disable)]


