---
uid: postsharp-components
redirect_from:
  - introduction/overview/postsharp-components
title: "PostSharp Components"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp offers components for Visual Studio, available as NuGet packages or a zip archive for teams not using NuGet. It includes build-time and run-time packages for various functions and aspects."
---
# PostSharp Components


## Visual Studio Tools for Metalama and PostSharp

This is the user interface of PostSharp. It extends the Visual Studio editor and provides a new menu, option pages, toolbox windows, diagnostics, code actions, and debugging enhancements.

For details regarding the installation of this component, see <xref:installation> and <xref:postsharp-unattended>. 


## NuGet packages

All build-time and run-time artifacts are released as NuGet packages. Build-time packages are required to build your projects, but only the content of run-time packages is required to execute your applications.

If you build NuGet packages that use PostSharp but do not define custom aspects, your package should only reference the relevant PostSharp run-time packages, not the build-time ones.

> [!NOTE]
> The PostSharp License Agreement refers to run-time packages as *redistributables*. The license agreement allows for royalty-free redistribution of run-time packages, but stricter conditions apply to the redistribution of build-time packages. 

The following table lists all PostSharp packages:

| Run-time package | Build-time package | Description |
|------------------------------------------------------|--------------------------------------------------------|-------------------------------------------------|
| *PostSharp.Redist* | *PostSharp* | PostSharp Framework. The build-time package includes the PostSharp compiler. |
| *PostSharp.Patterns.Common.Redist* | *PostSharp.Patterns.Common* | Common logic shared between pattern libraries. Code contracts. |
| *PostSharp.Patterns.Aggregation.Redist* | *PostSharp.Patterns.Aggregation* | Aggregatable and Disposable aspects. |
| *PostSharp.Patterns.Model.Redist* | *PostSharp.Patterns.Model* | NotifyPropertyChanged aspect. |
| *PostSharp.Patterns.Xaml.Redist* | *PostSharp.Patterns.Xaml* | Command, Dependency Property and Attached Property aspects. |
| *PostSharp.Patterns.Threading.Redist* | *PostSharp.Patterns.Threading* | Threading models, thread dispatching aspects. |
| *PostSharp.Patterns.Caching.Redist* | *PostSharp.Patterns.Caching* | Caching aspect. |
| *PostSharp.Patterns.Caching.Redis* | N/A | Redis connector for *PostSharp.Patterns.Caching*.  |
| *PostSharp.Patterns.Caching.Azure* | N/A | Azure connector for *PostSharp.Patterns.Caching*.  |
| *PostSharp.Patterns.Caching.IMemoryCache* | N/A | `IMemoryCache` connector for *PostSharp.Patterns.Caching*. |
| *PostSharp.Patterns.Diagnostics.Redist* | *PostSharp.Patterns.Diagnostics* | Logging aspect. |
| *PostSharp.Patterns.Diagnostics.ApplicationInsights* | N/A | Application Insights connector for *PostSharp.Patterns.Diagnostics*.  |
| *PostSharp.Patterns.Diagnostics.CommonLogging* | N/A | Common.Logging connector for *PostSharp.Patterns.Diagnostics*.  |
| *PostSharp.Patterns.Diagnostics.Log4Net* | N/A | Log4Net connector for *PostSharp.Patterns.Diagnostics*.  |
| *PostSharp.Patterns.Diagnostics.Microsoft* | N/A | Microsoft.Extensions.Logging connector for *PostSharp.Patterns.Diagnostics*.  |
| *PostSharp.Patterns.Diagnostics.NLog* | N/A | NLog connector for *PostSharp.Patterns.Diagnostics*.  |
| *PostSharp.Patterns.Diagnostics.Serilog* | N/A | Serilog connector for *PostSharp.Patterns.Diagnostics*.  |
| *PostSharp.Patterns.Diagnostics.Tracing* | N/A | System.Diagnostics connector for *PostSharp.Patterns.Diagnostics*.  |
| *PostSharp.Patterns.Diagnostics.Configuration* | N/A | Configuration of verbosity from a remote or local XML file. |
| *PostSharp.Patterns.Diagnostics.DiagnosticSource* | N/A | Collects events from <xref:System.Diagnostics.DiagnosticSource>.  |
| *PostSharp.Patterns.Diagnostics.HttpClient* | N/A | Collects events from <xref:System.Net.Http.HttpClient>.  |
| *PostSharp.Patterns.Diagnostics.AspNetCore* | N/A | Collects events from ASP.NET Core. |
| *PostSharp.Patterns.Diagnostics.AspNetFramework* | N/A | Collects events from ASP.NET on .NET Framework. |

## Package versions

The *PostSharp.Redist* package must have the same version as the *PostSharp* package. A project can get a different version of *PostSharp.Redist* when it references a package of a higher version that depends on *PostSharp.Redist*, such as a logging backend. PostSharp checks the two versions during the build:

* When *PostSharp.Redist* has a higher version than *PostSharp*, the build fails with an error. Reference the *PostSharp* package of the same version explicitly. To find which package requires the higher version of *PostSharp.Redist*, run `dotnet nuget why <project> PostSharp.Redist`.

* When *PostSharp.Redist* has a lower version than *PostSharp*, the build emits a warning. Versions of *PostSharp.Redist* earlier than 2027.0 do not perform this check. Reference the *PostSharp.Redist* package of the same version explicitly.

To skip this check, set the `PostSharpRedistVersionCheckDisabled` MSBuild property to `True`. See <xref:msbuild-properties>.

## See Also

**Other Resources**

<xref:deployment-end-user>
