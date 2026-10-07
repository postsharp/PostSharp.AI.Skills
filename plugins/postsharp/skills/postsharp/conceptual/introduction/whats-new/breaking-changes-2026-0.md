---
uid: breaking-changes-20260
redirect_from:
  - introduction/whatsnew/breaking-changes-20260
title: "Breaking Changes in PostSharp 2026.0"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 2026.0 introduces major breaking changes including discontinued support for .NET Framework versions older than 4.5.2 (4.7.1 for the pattern libraries), .NET Standard 1.3, and end-of-life .NET versions."
---
# Breaking Changes in PostSharp 2026.0

PostSharp 2026.0 consolidates our supported target platforms to simplify our build environment and streamline ongoing maintenance. This release focuses on actively maintained .NET versions while discontinuing support for legacy platforms that have reached end-of-life. Additionally, it officially removes the long-obsolete `[DeadlockDetectionPolicy]` feature and marks undo/redo for obsolescence.

We understand that breaking changes require effort to address, and we have carefully selected which platforms to discontinue with the goal of minimizing customer impact. Most applications targeting modern .NET versions (.NET 8.0 and later, .NET Framework 4.7.1 and later) will experience minimal or no impact from these changes.

## Deprecation of the deadlock detection feature

PostSharp 2026.0 removes the API for `[DeadlockDetectionPolicy]`. This feature has been marked `[Obsolete]` since 2024.0.

## Undo/redo marked obsolete

The `[Recordable]` aspect and related APIs are now marked as obsolete and should not be used. This feature has long been de facto obsolete due to lack of support for `async` methods and its low user base.

These APIs will not be maintained and will be removed in a future release.

## Deprecation of target frameworks

PostSharp 2026.0 removes support for most pre-2017 frameworks and .NET Core versions that are no longer supported by Microsoft. However, we have kept support for .NET 6.0 due to its widespread adoption as indicated by our telemetry data.

> [!WARNING]
> If you are affected by these deprecations and cannot update your target frameworks, remain on PostSharp 2024.0 LTS. Contact us to indicate your dependency on this version and potentially negotiate an extension of its support window.

The following table summarizes all target framework changes across PostSharp packages:

| Package | Discontinued TFM | Replacement TFM |
|---------|------------------|-----------------|
| PostSharp.Redist | .NET Framework 3.5 | .NET Framework 4.5.2 |
| PostSharp.Redist | .NET Framework 4.5 | .NET Framework 4.5.2 |
| PostSharp.Redist | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Redist | .NET 5.0 | .NET 6.0 |
| PostSharp.Redist | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Common.Redist | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Common.Redist | .NET Framework 4.6 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Common.Redist | .NET Framework 4.7 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Common.Redist | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Common.Redist | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Common.Redist | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Aggregation.Redist | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Aggregation.Redist | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Aggregation.Redist | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Aggregation.Redist | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Caching | .NET Framework 4.6.1 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Caching | .NET Framework 4.7 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Caching | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Caching | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Caching.Azure | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Caching.Azure | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Caching.IMemoryCache | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Caching.IMemoryCache | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Caching.Redis | .NET Framework 4.7 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Caching.Redis | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Caching.Redis | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.Redist | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Redist | .NET Framework 4.6 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Redist | .NET Framework 4.7 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Redist | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Diagnostics.Redist | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.Redist | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.ApplicationInsights | .NET Framework 4.5.2 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.ApplicationInsights | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.ApplicationInsights | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.AspNetCore | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.AspNetCore | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.AspNetFramework | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.CommonLogging | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.CommonLogging | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Diagnostics.CommonLogging | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.CommonLogging | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.Configuration | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Configuration | .NET Framework 4.6.1 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.DiagnosticSource | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.DiagnosticSource | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.DiagnosticSource | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.HttpClient | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.HttpClient | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.HttpClient | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.Log4Net | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Log4Net | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Diagnostics.Log4Net | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.Log4Net | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.Microsoft | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.Microsoft | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.NLog | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.NLog | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Diagnostics.NLog | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.NLog | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.Serilog | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Serilog | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Diagnostics.Serilog | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.Serilog | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Diagnostics.Tracing | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Tracing | .NET Framework 4.6 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Diagnostics.Tracing | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Diagnostics.Tracing | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Diagnostics.Tracing | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Model.Redist | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Model.Redist | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Model.Redist | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Model.Redist | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Threading.Redist | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Threading.Redist | .NET Standard 1.3 | .NET Standard 2.0 |
| PostSharp.Patterns.Threading.Redist | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Threading.Redist | .NET 7.0 | .NET 8.0 |
| PostSharp.Patterns.Xaml | .NET Framework 4.5 | .NET Framework 4.7.1 |
| PostSharp.Patterns.Xaml | .NET Core 3.0 | .NET 6.0 |
| PostSharp.Patterns.Xaml | .NET 5.0 | .NET 6.0 |
| PostSharp.Patterns.Xaml | .NET 7.0 | .NET 8.0 |

