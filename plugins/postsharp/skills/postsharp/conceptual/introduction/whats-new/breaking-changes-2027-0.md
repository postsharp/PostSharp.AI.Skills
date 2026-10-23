---
uid: breaking-changes-20270
title: "Breaking Changes in PostSharp 2027.0"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 2027.0 moves the supported .NET versions to .NET 10 and .NET 11, replaces the Windows options application, requires a license for interactive builds on Linux and macOS, switches on usage reporting by default, applies TreatWarningsAsErrors to the warnings of PostSharp, infers the groups of advices for each class, changes the caching aspects and the caching API, and no longer shares the dependencies of the compiler with add-ins."
---
# Breaking Changes in PostSharp 2027.0

This article lists the changes of PostSharp 2027.0 that can require a change in your projects, your build agents or your add-ins. For the new features, see <xref:whats-new-20270>.

## Supported platforms

### Supported .NET versions move to .NET 10 and .NET 11

.NET 6, .NET 8 and .NET 9 are out of Microsoft support when PostSharp 2027.0 ships. They are no longer supported target frameworks, and the packages no longer contain assets for them:

* A project that targets one of these frameworks uses the .NET Standard 2.0 asset of a package, when the package has one.
* `PostSharp.Patterns.Xaml` has no .NET Standard 2.0 asset, so a Windows Presentation Foundation application that targets .NET 8 or .NET 9 finds no compatible asset. The Windows-specific assets of `PostSharp.Patterns.Threading` have the same limitation.
* The .NET 10 SDK is the minimum SDK to build a project with PostSharp. The .NET 8 and .NET 9 SDKs are no longer supported.

Upgrade your projects to .NET 10 or .NET 11. The .NET Framework and .NET Standard 2.0 targets are not affected.

> [!WARNING]
> If you cannot upgrade your target frameworks, remain on PostSharp 2026.0 or PostSharp 2024.0 LTS.

## Tools, licensing and telemetry

### The Windows options application is replaced

The Windows options application (`PostSharp.Settings.UI.exe`) and its command line companion (`PostSharp.Settings.exe`) are removed. They are replaced by a browser-based user interface and by the `postsharp` command line tool, which work on Windows, Linux and macOS (see <xref:whats-new-20270>).

* On Windows, a build no longer opens dialogs. When a license is required, when a license or a trial is about to expire, or when the license server cannot be reached, PostSharp shows a toast notification after the build. The notification opens the corresponding page of the browser-based user interface.
* On Linux and macOS, a build shows no notification. Register a license key with `postsharp license register`, or supply it with the `PostSharpLicense` MSBuild property or in `postsharp.config` (see <xref:deploying-keys>).
* The build messages that report a missing or expiring license are unchanged.
* The `PostSharp.Settings.Common`, `PostSharp.Settings.AbstractUI` and `PostSharp.Settings.PostSharpIL` packages are no longer published.

### Interactive builds on Linux and macOS require a license

Until PostSharp 2026.0, every build on Linux and macOS was considered unattended, so it received the free license for build servers. PostSharp 2027.0 detects unattended builds in the same way on all platforms: a build that runs under a continuous integration agent, a service or a container is unattended, and a build that a developer starts from a terminal is not.

An interactive build on Linux or macOS therefore requires a license, as on Windows. Register a license key with `postsharp license register`, or supply it with the `PostSharpLicense` MSBuild property or in `postsharp.config`. Builds on build servers and on Windows are not affected.

### Usage reporting is switched on by default

PostSharp 2026.0 asked before it reported usage data. PostSharp 2027.0 switches usage reporting on at the first build, unless you opt out, and shows a notification. Crash reports are still sent only after you agree.

To opt out, set the `POSTSHARP_TELEMETRY_OPT_OUT` environment variable, or set the `TelemetryEnabled` property to `False` in the `postsharp.config` file at the root of the repository (see <xref:whats-new-20270>).

## Build

### The pipe server runs on Windows build agents

Until PostSharp 2026.0, PostSharp did not use the pipe server on an unattended build, so no PostSharp process remained after the build. PostSharp 2027.0 uses the pipe server wherever the `PostSharpUsePipeServer` property allows it. On a Windows build agent, the pipe server (`postsharp-x64-srv.exe`) therefore keeps running after the build, as on a developer's machine, and it locks the files it uses.

To restore the previous behavior, set `PostSharpUsePipeServer` to `False` on the build agent. To stop the pipe server at the end of a job, run `postsharp shutdown`. The `PostSharpAllowPipeServerWhenUnattended` property is obsolete, and setting it causes a build warning.

### .NET projects are compiled in a pipe server

Until PostSharp 2026.0, each compilation of a .NET project ran in a process of its own, which ended with the compilation. In PostSharp 2027.0, `dotnet build` compiles .NET and .NET Standard projects in the .NET pipe server (see <xref:whats-new-20270>). The server keeps running after the build, on developer machines and on build agents, on Windows, Linux and macOS, until it has had no build for 15 minutes. It keeps the environment variables of the build that started it: when you change a variable that PostSharp reads, stop the server first.

Since the server runs many compilations in one process, build-time code can now leak memory, which it could not do in a process of its own. When the build-time code of an aspect, or a static constructor or module initializer that it triggers, subscribes to a static event, starts a timer or registers something for the whole process, the whole compilation stays in memory, and the server uses more memory with each build. See <xref:in-process-memory-leaks> to avoid, detect and diagnose these leaks.

To compile each project in a process of its own, as before, set `PostSharpUsePipeServer` to `False`. To stop the server, for instance at the end of a job, run `postsharp shutdown`.

### TreatWarningsAsErrors applies to the warnings of PostSharp

Until PostSharp 2026.0, the `TreatWarningsAsErrors` and `WarningsAsErrors` MSBuild properties had no effect on the warnings of PostSharp and of aspects. In PostSharp 2027.0, `TreatWarningsAsErrors` escalates them to errors, except <xref:ps0131?text=PS0131>. A project that sets `TreatWarningsAsErrors` can therefore fail to build.

To keep the warnings of PostSharp as warnings, set the `PostSharpTreatWarningsAsErrors` property to `False`, or list the codes in `WarningsNotAsErrors`. See <xref:ignoring-warnings>.

## Aspect framework

### Advices are grouped by class, and SelfPointcut and Master are obsolete

PostSharp 2027.0 infers the groups of the advices of each class of an aspect: an advice without pointcut and without <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> joins the group of its class that has no advice of its kind yet (see <xref:pointcuts>). This changes the aspects that relied on `[SelfPointcut]` to separate advices:

* Several advices of different kinds that each had `[SelfPointcut]`, for instance an `OnEntry` and an `OnExit` advice, were separate groups. They are now one group, so they share the <xref:PostSharp.Aspects.MethodExecutionArgs> object and its <xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag>. An `OnExit` advice can now read the tag that the `OnEntry` advice set, and the advices can overwrite each other's tag.
* `[SelfPointcut]` and the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property are obsolete, so the C# compiler reports their use with the warning CS0618. `[SelfPointcut]` is ignored. <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> keeps its meaning; replace it with <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group>, and set the same name on all the advices of the group.
* PostSharp reports new warnings: <xref:la0255?text=LA0255> for a <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> property that selects the group that PostSharp infers anyway, and <xref:la0257?text=LA0257> for two groups of one family that apply to the target of the aspect.

Since `TreatWarningsAsErrors` now applies to the warnings of PostSharp, these warnings can fail the build.

When the advices must stay in separate groups, give each of them a <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> name of its own. The groups then both apply to the target of the aspect, so PostSharp reports <xref:la0257?text=LA0257>, which you can disable for this aspect (see <xref:ignoring-warnings>). When PostSharp also reports <xref:ps0114?text=PS0114> because the groups are not ordered, order them with <xref:PostSharp.Aspects.Dependencies.AdviceDependencyAttribute>.

### New warning LA0250 for OnYield and OnResume advices of async methods

PostSharp 2027.0 can let an exception thrown by the `OnYield` or `OnResume` advice of an `async` method behave as an exception of the `await` expression: the aspects that suspended the method resume first, then the `catch` and `finally` blocks of the method around the `await` see the exception. If the method does not catch it, the `OnException` and `OnExit` advices run and the task faults. If the method catches it, the method continues.

This behavior applies only when the new `PostSharpCompatibilityLevel` MSBuild property is `2027.0` or higher. When the property is not set, the behavior of PostSharp 2026.0 is kept, and PostSharp emits the warning <xref:la0250?text=LA0250> once per aspect type that has an `OnYield` or `OnResume` advice applied to an `async` method, including the aspects of the Pattern Libraries. Since `TreatWarningsAsErrors` now applies to the warnings of PostSharp, this warning can fail the build.

To remove the warning, set `PostSharpCompatibilityLevel` to `2027.0` to adopt the new behavior, or to `2026.0` to keep the previous one, in the project or in a `Directory.Build.props` file:

```xml
<PropertyGroup>
  <PostSharpCompatibilityLevel>2027.0</PostSharpCompatibilityLevel>
</PropertyGroup>
```

With the new behavior, an aspect must tolerate that the method continues after its `OnYield` or `OnResume` advice throws, because the user code can catch the exception. See <xref:semantic-advising>.

### CurrentTask is null before the first yield

`[CurrentTask]` now gives the task that the caller of the async method observes. This task exists only once the method has yielded, so the parameter is `null` in the entry advice, in the first yield advice, and when the method completes without yielding. Before, it could give a task that was not the caller's task and never completed. This change affects only advices with bound parameters, which were not documented before PostSharp 2027.0.

## Pattern Libraries

### Caching aspects and the caching services

PostSharp Caching 2027.0 introduces caching services (see <xref:caching-dependency-injection>). The following changes can affect existing code. The C# compiler reports the use of the obsolete members with the warning CS0618, which fails the build of a project that treats warnings as errors.

* <xref:PostSharp.Patterns.Caching.CachingServices.DefaultBackend> and <xref:PostSharp.Patterns.Caching.CachingServices.DefaultKeyBuilder> are obsolete. Use the <xref:PostSharp.Patterns.Caching.ICachingService.Backend> and <xref:PostSharp.Patterns.Caching.ICachingService.KeyBuilder> properties of <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>, to which they are now aliases.
* The overloads of <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation> that take a dependency (a string, an <xref:PostSharp.Patterns.Caching.Dependencies.ICacheDependency> or another object) or a <xref:System.Reflection.MethodInfo> are obsolete. Call the <xref:PostSharp.Patterns.Caching.ICachingService.Invalidate*> methods of <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService> instead. The overloads that take a delegate are not obsolete.
* The nested type `CachingServices.CachingProfileRegistry` is obsolete, and replaced by <xref:PostSharp.Patterns.Caching.CachingProfileRegistry>. <xref:PostSharp.Patterns.Caching.CachingServices.Profiles> keeps its type, so assemblies compiled against a previous version still run.
* <xref:PostSharp.Patterns.Caching.CacheAttribute> and <xref:PostSharp.Patterns.Caching.InvalidateCacheAttribute> now derive from <xref:PostSharp.Aspects.MethodLevelAspect>, are sealed, and no longer implement <xref:PostSharp.Aspects.IMethodInterceptionAspect>. Their `OnInvoke` and `OnInvokeAsync` methods have new signatures. Code that derives from these classes or calls these methods must be changed.
* An async iterator, or an async method that returns `ValueTask<T>`, with <xref:PostSharp.Patterns.Caching.CacheAttribute> and the default <xref:PostSharp.Patterns.Caching.CacheAttribute.UnsupportedTargetAction> now causes the error <xref:cac013?text=CAC013> instead of <xref:la0197?text=LA0197>.
* Each caching service has its own caching profiles. <xref:PostSharp.Patterns.Caching.CachingServices.Profiles> configures only the methods that use <xref:PostSharp.Patterns.Caching.CachingServices.DefaultService>.
* With <xref:PostSharp.Patterns.Caching.CacheConfigurationAttribute.UseDependencyInjection> set to `true`, the public constructors of the classes with cached methods receive a new parameter. This is a binary breaking change for the consumers of these classes, but it applies only to the code that opts in.

### Architecture constraints are enforced in more cases

The architecture constraints report the following uses, which PostSharp 2026.0 did not report (see <xref:control-visibility> and <xref:control-implementation>):

* A type that implements an interface of `PostSharp.dll` marked with <xref:PostSharp.Constraints.InternalImplementAttribute> causes the warning <xref:ar0101?text=AR0101>.
* Applying a custom attribute marked with <xref:PostSharp.Constraints.InternalAttribute> in another assembly causes the warning <xref:ar0105?text=AR0105>.
* <xref:PostSharp.Constraints.InternalAttribute> now reads `InternalsVisibleTo` in the right direction. A friend assembly of the declaring assembly can use the element without a warning. An assembly that only grants access to its own internals gets the warning <xref:ar0105?text=AR0105>.

### Serilog backend

The `PostSharp.Patterns.Diagnostics.Serilog` package now depends on Serilog 4.4.0 instead of Serilog 2.3.0:

* An application that uses Serilog 2 or Serilog 3 is upgraded to Serilog 4.4.0 or later. Sinks and extensions built for Serilog 2 may need a newer version.
* On .NET Framework, an application that loads another version of Serilog needs a binding redirect, which the .NET SDK usually generates.

The Serilog property of a string parameter, return value or property value no longer contains quotes. For instance, the property `Arg1` is `guest`, not `"guest"`. In the rendered message, Serilog adds the quotes. See <xref:serilog>.

## Add-ins

### Add-ins compile against PostSharp.Sdk

The `PostSharp.Compiler.Common` and `PostSharp.Compiler.Engine` packages no longer contain assemblies. They depend on the new `PostSharp.Sdk` package, which contains .NET Standard 2.0 reference assemblies of the compiler. This change affects only the authors of PostSharp add-ins:

* Reference `PostSharp.Sdk` instead of `PostSharp.Compiler.Engine` or `PostSharp.Compiler.Common`.
* Use only the public API of the reference assemblies.
* Do not declare a type that derives from a C# record of the compiler or of its dependencies. Such a type fails to load on .NET.
* The compiler no longer shares its third-party dependencies, such as Newtonsoft.Json, System.Text.Json, System.Collections.Immutable or Microsoft.Extensions.DependencyInjection, with the code it loads. An add-in loaded with `<Using>` in `postsharp.config` that relied on the compiler to supply one of these assemblies must ship it beside the add-in.
