---
uid: msbuild-properties
redirect_from:
  - deploymentconfiguration/configuration/msbuild-properties
title: "Well-Known MSBuild Properties"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on well-known MSBuild properties in PostSharp, including general, hosting, diagnostic, directory, and other properties. Each property is described in a table format."
---
# Well-Known MSBuild Properties

> [!NOTE]
> The integration of PostSharp with MSBuild is implemented in files *PostSharp.tasks* and *PostSharp.targets*. These files define some properties and items that are not documented here. They are considered implementation details and may change without notice. 

> [!NOTE]
> You can set any of these properties with an environment variable of the same name. PostSharp also reads some environment variables that have no equivalent MSBuild property. See <xref:environment-variables>.


## General Properties

The following properties are most commonly overwritten. They can also be edited in Visual Studio using the PostSharp project property page.

| Property name | Description |
|---------------|-------------|
| `PostSharpSearchPath` | A semicolon-separated list of directories added to the PostSharp search path. PostSharp will probe these directories when looking for an assembly or an add-in. Note that several directories are automatically added to the search path: the .NET Framework reference directory, the directories containing the dependencies of your project and the directories added to the reference path of your project (tab **Reference Path** in the Visual Studio project properties dialog box).  |
| `SkipPostSharp` | `True` if PostSharp should not be executed.  |
| `PostSharpDisabledMessages` | Comma-separated list of warnings and messages that should be ignored. |
| `PostSharpEscalatedMessages` | Comma-separated list of warnings that should be escalated to errors. Use `*` to escalate all warnings.  |
| `PostSharpTreatWarningsAsErrors` | When set to `True`, PostSharp escalates all its warnings to errors, except <xref:ps0131?text=PS0131> and the codes listed in `WarningsNotAsErrors`. The default value is the value of `TreatWarningsAsErrors`. PostSharp always escalates the PostSharp codes listed in `WarningsAsErrors`. See <xref:ignoring-warnings>. |
| `PostSharpLicense` | License key or URL of the license server. |
| `PostSharpProperties` | Additional properties passed to the PostSharp project, in format `Name1=Value1;Name2=Value2`. See <xref:configuration-postsharp>.  |
| `PostSharpCompatibilityLevel` | Version of PostSharp, for instance `2026.0`, whose behavior PostSharp keeps where a later version changed it. Since PostSharp 2027.0, it determines the path of an exception thrown by the `OnYield` or `OnResume` advice of an `async` method: at `2027.0` or higher, the exception flows through the method as an exception of the `await` expression; below, or when the property is not set, it keeps the behavior of PostSharp 2026.0, and an unset property causes the warning <xref:la0250?text=LA0250>. See <xref:semantic-advising>. |
| `PostSharpConstraintVerificationEnabled` | Determines whether verification of architecture constraints is enabled. The default value is `True`.  |


## Hosting Properties

PostSharp both reads and executes assemblies that it transforms. It needs to run under a runtime that allows execution of the input assembly and of its dependencies. This is done automatically based on the target framework of the project being built. The following properties allow you to influence the default behavior.

| Property name | Description |
|---------------|-------------|
| `PostSharpUsePipeServer` | Specifies whether PostSharp should use a background process invoked synchronously from MSBuild. Valid values are `True` and `False`. The default value is `True` when the PostSharp process runs on x64 or ARM64, and `False` when it runs on x86. Using the pipe server results in lower build time, since PostSharp would otherwise have to be started every time a project is built. For a .NET Framework project, the pipe server uses native code and the CLR Hosting API to control the way assemblies are loaded in application domains; the assembly loading algorithm is generally more accurate and predictable than with the managed host. For a .NET project, the property selects the .NET pipe server, which only `dotnet build` and the other commands of the .NET SDK use; the MSBuild of Visual Studio runs the compiler in a process of its own. A compilation for x86 never uses a pipe server: a 32-bit process does not have enough address space for a server. |
| `PostSharpVerifyLoadContextUnload` | `True` makes the .NET pipe server check, after each compilation, that the compilation is unloaded, and report error PS0286 when it is still in memory 5 seconds later. The default value is `False`. See <xref:in-process-memory-leaks>. |
| `PostSharpTargetProcessor` | Processor architecture of the PostSharp hosting process. Valid values are `x86`, `x64` and `arm64`. Since PostSharp needs to execute the current project during the build, the processor architecture of the PostSharp process must be compatible with the target platform of the current project. By default, PostSharp uses the target platform of the current project. If the project targets `AnyCPU`, PostSharp uses the architecture of the operating system. <br>This property is valid only when targeting .NET Framework, otherwise it is ignored. |
| `PostSharpAllowPipeServerWhenUnattended` | Obsolete since PostSharp 2027.0, where setting it causes a build warning: PostSharp uses the pipe server in unattended builds too, and only `PostSharpUsePipeServer` decides whether the pipe server is used. From PostSharp 2026.0.17 until 2027.0, `True` makes PostSharp use the pipe server in an unattended build, for instance on a build server or in a container, and the default value is `False`. Before PostSharp 2026.0.17, PostSharp never uses the pipe server in an unattended build. |
| `PostSharpPipeTimeout` | The time, in milliseconds, during which the PostSharp MSBuild task waits for the pipe server. The task waits when the pipe server is busy with the builds of other projects, and when it opens the connection to the pipe server. When this time expires, the task stops waiting and reports a warning. The default value is `600000` (10 minutes). <br>This property is valid only when the pipe server is used, otherwise it is ignored. |
| `PostSharpMutexTimeout` | The time, in milliseconds, during which the PostSharp MSBuild task waits for a machine-wide lock. PostSharp uses these locks to serialize the access of concurrent builds to its shared directories, for instance when it restores its build-time dependencies, or generates the native images and the *deps.json* and *runtimeconfig.json* files of its process. The default value is `600000` (10 minutes). |
| `PostSharpBuild` | Build configuration of PostSharp. Valid values are `Release` and `Debug`. Only the `Release` build is distributed in the normal PostSharp packages.  |
| `PostSharpHostConfigurationFile` | Item group of configuration files containing assembly binding redirections that should be taken into account by the PostSharp hosting process. When targeting .NET Framework, it includes by default a file that PostSharp generates from the `app.config` file of the project and from the binding redirects suggested by MSBuild.  |
| `PostSharpDependencyRestoreDisabled` | Specifies whether PostSharp should disable restore of its build-time dependencies. Valid values are `True` and `False`. <br>PostSharp's .NET Core process requires its build-time dependency NuGet packages to be present on the machine. By default it uses NuGet restore to download all packages into a fallback directory. You can use this property to disable this process, at which point PostSharp will expect all dependencies to be present in the NuGet package cache directory used to restore the project being built. You can find the list of dependencies in the verbose build log.<br>This property is valid only when targeting .NET Standard and .NET Core, otherwise it is ignored. |
| `PostSharpDependencyRestoreProperties` | MSBuild properties to set in the projects that PostSharp generates to restore its build-time dependencies, as semicolon-separated `Name=Value` pairs, for example `NuGetAudit=false`. <br>PostSharp generates these projects outside of your source tree, so your `Directory.Build.props` files do not apply to them. This property affects only these projects, and not the build of your project. <br>Write a semicolon in a value as `%3B`. The value is evaluated in the generated project, so write `%24(NoWarn)` to refer to a property of that project: for example, `NoWarn=%24(NoWarn)%3BNU1901` adds `NU1901` to the warnings that the restore ignores. You cannot set `TargetFramework`, `TargetFrameworks` or `RestoreOutputPath`. <br>This property is valid only when targeting .NET Standard and .NET Core, otherwise it is ignored. It is available since PostSharp 2027.0. |
| `PostSharpReadyToRunDisabled` | Specifies whether PostSharp should try to generate and use ReadyToRun images of itself. Valid values are `True` and `False`, the default value is `False`. <br>When the build runs under .NET Core SDK 3.1 and later, the PostSharp .NET Core process attempts to generate and use ReadyToRun images to improve build-time performance. If generating these images fails, multiple build warnings may appear in the build log.<br>This property is valid only when targeting .NET Standard and .NET Core, otherwise it is ignored. |


## Diagnostic Properties

| Property name | Description |
|---------------|-------------|
| `PostSharpAttachDebugger` | If this property is set to `True`, PostSharp will break before starting execution, allowing you to attach a debugger to the PostSharp process. The default value is `False`. For details, see <xref:debugging-buildtime>. <br>When targeting .NET Framework, PostSharp will invoke `System.Diagnostics.Debugger.Launch()`. When targeting .NET Core or .NET Standard, PostSharp will print its current process ID into the build log and wait for the debugger to be attached.  |
| `PostSharpTrace` | A semicolon-separated list of trace categories to be enabled, or `*` to enable all categories. In PostSharp 2026.0.16 and earlier, `*` is not supported, and causes a `TypeInitializationException` when the pipe server is not used. Most commonly used categories are `AssemblyBinder`, `PlatformContext`, `Domain`, `ReflectionBinding`, `ProjectLoader` and `FileLock` (since PostSharp 2026.0.19). To see the trace output in the build log, set the verbosity to at least `normal`.  |
| `PostSharpBenchmark` | A semicolon-separated list of benchmark categories to be enabled or `all` for all benchmarks to be enabled.  |
| `PostSharpBenchmarkOutputFile` | An absolute path to a CSV file where benchmark data should be appended. If the file does not exist, it is created with a header line. This is useful when benchmarking multiple projects at once or in automated builds. |
| `PostSharpExpectedMessages` | A semicolon-separated list of codes of expected messages. PostSharp will return a failure code if any expected message was not emitted. This property is used in unit tests of aspects, to ensure that the application of an aspect results in the expected error message. |
| `PostSharpIgnoreError` | If this property is set to `True`, the `PostSharp` MSBuild task will succeed even if PostSharp returns an error code, allowing the build process to continue. The project or targets file can check the value of the `ExitCode` output property of the `PostSharp` MSBuild task to take action.  |
| `PostSharpFailOnUnexpectedMessage` | This property should be used jointly with `PostSharpExpectedMessages`. If it is set to `True`, PostSharp will fail if any unexpected message was emitted, even if this message was not an error. This property is used in unit tests of aspects, to ensure that the application of an aspect did not result in other messages than expected.  |


## Directory Properties

PostSharp uses multiple directories at build time. You can use the following properties to override default locations.

| Property name | Description |
|---------------|-------------|
| `PostSharpBinaryDirectory` | Location of the PostSharp binaries. PostSharp extracts its binaries into one subdirectory of this directory per PostSharp version.<br>Default on Windows is `%LOCALAPPDATA%\PostSharp`. Default on UNIX systems is `$XDG_DATA_HOME/postsharp`, or `~/.local/share/postsharp` if `XDG_DATA_HOME` is not set.  |
| `PostSharpCacheDirectory` | Location of the PostSharp caches. PostSharp uses multiple build-time caches shared between PostSharp versions to improve its performance.<br>Default on Windows is `%LOCALAPPDATA%\PostSharp`. Default on UNIX systems is the `cache` subdirectory of the default binary directory.  |
| `PostSharpDependencyDirectory` | Location of the PostSharp dependency directory. PostSharp uses this directory to store its build-time dependencies, usually shared between versions.<br>Default on Windows is `%LOCALAPPDATA%\PostSharp\NuGetFallback`. Default on UNIX systems is the `NuGetFallback` subdirectory of the default binary directory.  |
| `PostSharpTempDirectory` | Location of the PostSharp temp directory. PostSharp uses this directory to store files that are temporary and do not have to be persisted.<br>Default on Windows is `%TEMP%\PostSharp`. Default on UNIX systems is the `temp` subdirectory of the default binary directory.  |


## Other Properties

| Property name | Description |
|---------------|-------------|
| `PostSharpProject` | Location of the PostSharp project (`*.psproj`) to be executed by PostSharp. By default, PostSharp uses the file `<MyProject>.psproj` of the project directory, if it exists. To disable PostSharp, use the `SkipPostSharp` property instead.  |
| `PostSharpRedistVersionCheckDisabled` | When set to `True`, the build does not check that the *PostSharp.Redist* package has the same version as the *PostSharp* package. By default, the build fails with an error when *PostSharp.Redist* has a higher version, and emits a warning when it has a lower version. See <xref:postsharp-components>. |
| `PostSharpLockedFileTimeout` | The time, in milliseconds, during which the PostSharp MSBuild targets retry to move or copy a file that another process locks, before they report an error. These targets move the output of the compiler before PostSharp runs, and copy the companion files after. The default value is `60000`. To configure the retries of the PostSharp compiler itself, see <xref:environment-variables>.  |
| `PostSharpLockedFileWarningTimeout` | The time, in milliseconds, after which the PostSharp MSBuild targets report a warning for a file that another process locks. The default value is `250`.  |
| `PostSharpUseHardLink` | Use hard links instead of file copies when creating the snapshot for Visual Studio Code Analysis (FxCop). This property is `True` by default.  |
| `ExecuteCodeAnalysisOnPostSharpOutput` | When set to `True`, executes Microsoft Code Analysis on the *output* of PostSharp. By default, the analysis is done on the *input* of PostSharp, that is, on the output of the compiler. This property has no effect when Microsoft Code Analysis is disabled for the current build.  |
| `PostSharpCopyCodeAnalysisDependenciesDisabled` | When set to `True`, PostSharp will not copy all dependencies of the current project output into the *obj\Debug\Before-PostSharp* directory, which contains the copy of the assembly on which Microsoft Code Analysis is executed by default. This property has no effect when Microsoft Code Analysis is disabled for the current build or when the `ExecuteCodeAnalysisOnPostSharpOutput` property has been set to `True`.  |


