---
uid: breaking-changes-20240
redirect_from:
  - introduction/whatsnew/breaking-changes-20240
title: "Breaking Changes in PostSharp 2024.0"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 2024.0 introduces several breaking changes including build errors, changes in behavior, and deprecated platforms and features."
---
# Breaking Changes in PostSharp 2024.0

PostSharp 2024.0 contains a larger number of breaking changes mainly related to platform upgrades and dependency security.

## PostSharp
* To build .NET Standard, .NET Core and .NET projects, .NET 6.0 SDK is required. Older SDKs are no longer supported.
* By default, when building .NET Framework projects, the x64 version of PostSharp Compiler will be used. 
  Projects referencing x86-specific libraries from projects targeting AnyCPU (default setting) should explicitly set the `PostSharpTargetProcessor` MSBuild property to `x86`.
* When building .NET Framework projects with x86 version of PostSharp Compiler, pipe-server is no longer used as default. This may lead to longer build times.
  If the use of pipe-server is necessary, the `PostSharpUsePipeServer` MSBuild property should be explicitly set to `true`.
* PostSharp aspects are now strongly encouraged to use `[PSerializable]` instead of `[Serializable]` for aspects serialization. 
  Projects that still require using `[Serializable]` for aspects should explicitly set the `PostSharpBinaryFormatterAllowed` MSBuild property to `True`.
* PostSharp Compiler internal dependencies have been upgraded. Consequently, downloading NuGet packages will be necessary during the initial build, potentially requiring manual action for users.

## PostSharp.Patterns.Common
* The dependency on `System.Text.RegularExpressions` has been upgraded to 4.3.1.

## PostSharp.Patterns.Caching
* The default serializer has been updated from `BinarySerializer` to `JsonCachingSerializer`. For persisted backends, this change implies that values are incompatible with older versions of the library, and it's advisable not to mix versions.
* `BinarySerializer` is now considered obsolete and will produce a warning.
* The package has a new dependency on the `System.Text.Json` 6.0.0.
* Support for targeting .NET Core 3.0 and earlier has been discontinued.
* Support for targeting .NET Framework 4.6.0 and earlier has been discontinued.

## PostSharp.Patterns.Caching.Azure
* `AzureCacheInvalidator` and `AzureCacheInvalidator2` have been replaced with <xref:PostSharp.Patterns.Caching.Backends.Azure.AzureServiceBusCacheInvalidator>, which utilizes the latest Azure API.
* The package has a new dependency on `Azure.Messaging.ServiceBus` 7.16.1 instead of `WindowsAzure.ServiceBus` and `Microsoft.Azure.ServiceBus`.
* The package has a new dependency on `Azure.Identity` 1.10.4.
* Support for targeting .NET Framework 4.7.0 and earlier has been discontinued.
* Support for targeting .NET Core 3.0 and earlier has been discontinued.

## PostSharp.Patterns.Diagnostics.AspNetCore
* The dependency on `Microsoft.AspNetCore.Mvc.Core` has been upgraded to 2.1.0.

## PostSharp.Patterns.Diagnostics.HttpClient
* The dependency on `System.Net.Http` has been upgraded to 4.3.4.

## PostSharp.Patterns.Diagnostics.ApplicationInsights
* The dependency on `Microsoft.ApplicationInsights` has been upgraded to 2.21.0.
* Support for targeting .NET Standard 1.6 and earlier has been discontinued.
* Support for targeting .NET Framework 4.5.1 and earlier has been discontinued.

## PostSharp.Patterns.Diagnostics.Log4Net
* The dependency on `log4net` has been upgraded to 2.0.12.

## PostSharp.Patterns.Diagnostics.Microsoft
* Support for targeting .NET Standard 1.6 and earlier has been discontinued.
* Support for targeting .NET Framework 4.7.0 and earlier has been discontinued.

## PostSharp.Patterns.Diagnostics.Serilog
* The dependency on `Serilog` has been upgraded for .NET Framework to 2.3.0 and it's now unified across all target frameworks.

## PostSharp.Patterns.Threading
* `DeadlockDetectionPolicy` is now considered obsolete and will not be maintained.