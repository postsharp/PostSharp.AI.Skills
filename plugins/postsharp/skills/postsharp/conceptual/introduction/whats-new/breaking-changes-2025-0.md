---
uid: breaking-changes-20250
redirect_from:
  - introduction/whatsnew/breaking-changes-20250
title: "Breaking Changes in PostSharp 2025.0"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 2025.0 introduces several breaking changes and deprecates platforms and features."
---
# Breaking Changes in PostSharp 2025.0

PostSharp 2025.0 contains a small number of breaking changes mainly related to platform upgrades and dependency security.

## PostSharp
* To build .NET Standard, .NET Core and .NET projects, .NET 8.0 SDK is required. Older SDKs are no longer supported.
* PostSharp aspects must now use `[PSerializable]` instead of `[Serializable]` for aspects serialization. 
  Projects that still require using `[Serializable]` for aspects should explicitly set the `PostSharpBinaryFormatterAllowed` MSBuild property to `True`, but the use of this option is unsupported, which means it is not guaranteed to work.
* PostSharp Compiler internal dependencies have been upgraded. Consequently, downloading NuGet packages will be necessary during the initial build, potentially requiring manual action for users.

## PostSharp.Patterns.Common
* The package has a new direct dependency on `System.Net.Http` 4.3.4.

## PostSharp.Patterns.Caching
* The package has a new direct dependency on `System.Text.Encodings.Web` 4.7.2.

## PostSharp.Patterns.Caching.Azure
* The dependency on `Azure.Core` has been upgraded to 1.38.0.
* The dependency on `Azure.Identity` has been upgraded to 1.11.4.

## PostSharp.Patterns.Diagnostics.AspNetCore
* The dependency on `Microsoft.AspNetCore.Http` has been upgraded to 2.1.22.
* The package has a new direct dependency on `Newtonsoft.Json` 13.0.1.
* The package has a new direct dependency on `System.Text.Encodings.Web` 4.7.2.

## PostSharp.Patterns.Diagnostics.ApplicationInsights
* The dependency on `Microsoft.ApplicationInsights` has been upgraded to 2.22.0.

## PostSharp.Patterns.Model
* The package no longer supports targeting `netstandard1.3` target framework.