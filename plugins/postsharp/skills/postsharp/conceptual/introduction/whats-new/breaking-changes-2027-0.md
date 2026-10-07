---
uid: breaking-changes-20270
title: "Breaking Changes in PostSharp 2027.0"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 2027.0 moves the supported .NET versions to .NET 10 and .NET 11, replaces the Windows options application, requires a license for interactive builds on Linux and macOS, switches on usage reporting by default, and applies TreatWarningsAsErrors to the warnings of PostSharp."
---
# Breaking Changes in PostSharp 2027.0

This article lists the changes of PostSharp 2027.0 that can require a change in your projects, your build agents or your add-ins. For the new features, see <xref:whats-new-20270>.

## Supported .NET versions move to .NET 10 and .NET 11

.NET 6, .NET 8 and .NET 9 are out of Microsoft support when PostSharp 2027.0 ships. They are no longer supported target frameworks, and the packages no longer contain assets for them:

* A project that targets one of these frameworks uses the .NET Standard 2.0 asset of a package, when the package has one.
* `PostSharp.Patterns.Xaml` has no .NET Standard 2.0 asset, so a Windows Presentation Foundation application that targets .NET 8 or .NET 9 finds no compatible asset. The Windows-specific assets of `PostSharp.Patterns.Threading` have the same limitation.
* The .NET 10 SDK is the minimum SDK to build a project with PostSharp. The .NET 8 and .NET 9 SDKs are no longer supported.

Upgrade your projects to .NET 10 or .NET 11. The .NET Framework and .NET Standard 2.0 targets are not affected.

> [!WARNING]
> If you cannot upgrade your target frameworks, remain on PostSharp 2026.0 or PostSharp 2024.0 LTS.

## The Windows options application is replaced

The Windows options application (`PostSharp.Settings.UI.exe`) and its command line companion (`PostSharp.Settings.exe`) are removed. They are replaced by a browser-based user interface and by the `postsharp` command line tool, which work on Windows, Linux and macOS (see <xref:whats-new-20270>).

* On Windows, a build no longer opens dialogs. When a license is required, when a license or a trial is about to expire, or when the license server cannot be reached, PostSharp shows a toast notification after the build. The notification opens the corresponding page of the browser-based user interface.
* On Linux and macOS, a build shows no notification. Register a license key with `postsharp license register`, or supply it with the `PostSharpLicense` MSBuild property or in `postsharp.config` (see <xref:deploying-keys>).
* The build messages that report a missing or expiring license are unchanged.
* The `PostSharp.Settings.Common`, `PostSharp.Settings.AbstractUI` and `PostSharp.Settings.PostSharpIL` packages are no longer published.

## Interactive builds on Linux and macOS require a license

Until PostSharp 2026.0, every build on Linux and macOS was considered unattended, so it received the free license for build servers. PostSharp 2027.0 detects unattended builds in the same way on all platforms: a build that runs under a continuous integration agent, a service or a container is unattended, and a build that a developer starts from a terminal is not.

An interactive build on Linux or macOS therefore requires a license, as on Windows. Register a license key with `postsharp license register`, or supply it with the `PostSharpLicense` MSBuild property or in `postsharp.config`. Builds on build servers and on Windows are not affected.

## Usage reporting is switched on by default

PostSharp 2026.0 asked before it reported usage data. PostSharp 2027.0 switches usage reporting on at the first build, unless you opt out, and shows a notification. Crash reports are still sent only after you agree.

To opt out, set the `POSTSHARP_TELEMETRY_OPT_OUT` environment variable, or set the `TelemetryEnabled` property to `False` in the `postsharp.config` file at the root of the repository (see <xref:whats-new-20270>).

## The pipe server runs on Windows build agents

Until PostSharp 2026.0, PostSharp did not use the pipe server on an unattended build, so no PostSharp process remained after the build. PostSharp 2027.0 uses the pipe server wherever the `PostSharpUsePipeServer` property allows it. On a Windows build agent, the pipe server (`postsharp-x64-srv.exe`) therefore keeps running after the build, as on a developer's machine, and it locks the files it uses.

To restore the previous behavior, set `PostSharpUsePipeServer` to `False` on the build agent. To stop the pipe server at the end of a job, run `postsharp shutdown`. The `PostSharpAllowPipeServerWhenUnattended` property is obsolete, and setting it causes a build warning.

## TreatWarningsAsErrors applies to the warnings of PostSharp

Until PostSharp 2026.0, the `TreatWarningsAsErrors` and `WarningsAsErrors` MSBuild properties had no effect on the warnings of PostSharp and of aspects. In PostSharp 2027.0, `TreatWarningsAsErrors` escalates them to errors, except `PS0131`. A project that sets `TreatWarningsAsErrors` can therefore fail to build.

To keep the warnings of PostSharp as warnings, set the `PostSharpTreatWarningsAsErrors` property to `False`, or list the codes in `WarningsNotAsErrors`. See <xref:ignoring-warnings>.

## Add-ins compile against PostSharp.Sdk

The `PostSharp.Compiler.Common` and `PostSharp.Compiler.Engine` packages no longer contain assemblies. They depend on the new `PostSharp.Sdk` package, which contains .NET Standard 2.0 reference assemblies of the compiler. This change affects only the authors of PostSharp add-ins:

* Reference `PostSharp.Sdk` instead of `PostSharp.Compiler.Engine` or `PostSharp.Compiler.Common`.
* Use only the public API of the reference assemblies.
* Do not declare a type that derives from a C# record of the compiler or of its dependencies. Such a type fails to load on .NET.

## Architecture constraints are enforced in more cases

The architecture constraints report the following uses, which PostSharp 2026.0 did not report (see <xref:control-visibility> and <xref:control-implementation>):

* A type that implements an interface of `PostSharp.dll` marked with <xref:PostSharp.Constraints.InternalImplementAttribute> causes the warning AR0101.
* Applying a custom attribute marked with <xref:PostSharp.Constraints.InternalAttribute> in another assembly causes the warning AR0105.
* <xref:PostSharp.Constraints.InternalAttribute> now reads `InternalsVisibleTo` in the right direction. A friend assembly of the declaring assembly can use the element without a warning. An assembly that only grants access to its own internals gets the warning AR0105.

## Serilog backend

The `PostSharp.Patterns.Diagnostics.Serilog` package now depends on Serilog 4.4.0 instead of Serilog 2.3.0:

* An application that uses Serilog 2 or Serilog 3 is upgraded to Serilog 4.4.0 or later. Sinks and extensions built for Serilog 2 may need a newer version.
* On .NET Framework, an application that loads another version of Serilog needs a binding redirect, which the .NET SDK usually generates.

The Serilog property of a string parameter, return value or property value no longer contains quotes. For instance, the property `Arg1` is `guest`, not `"guest"`. In the rendered message, Serilog adds the quotes. See <xref:serilog>.

## CurrentTask is null before the first yield

`[CurrentTask]` now gives the task that the caller of the async method observes. This task exists only once the method has yielded, so the parameter is `null` in the entry advice, in the first yield advice, and when the method completes without yielding. Before, it could give a task that was not the caller's task and never completed. This change affects only advices with bound parameters, which were not documented before PostSharp 2027.0.
