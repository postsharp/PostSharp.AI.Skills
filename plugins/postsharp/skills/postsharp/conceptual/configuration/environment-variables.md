---
uid: environment-variables
redirect_from:
  - deploymentconfiguration/configuration/environment-variables
title: "Well-Known Environment Variables"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document lists the environment variables that PostSharp reads directly, including the variables that throttle concurrent builds, configure the retries of locked files, control licensing and usage reporting, and enable diagnostics."
---
# Well-Known Environment Variables

PostSharp reads two kinds of environment variables:

* Environment variables named like an MSBuild property. MSBuild imports every environment variable as a property, so you can set any property of <xref:msbuild-properties> with an environment variable of the same name. For instance, the `PostSharpLicense` environment variable sets the `PostSharpLicense` property. For details, see <xref:configuration-msbuild>.
* Environment variables that PostSharp reads directly. They have no equivalent MSBuild property. Most of them are machine-wide settings, typically defined on a build server. This article lists them.

> [!NOTE]
> PostSharp can run in a background process named the pipe server (see the `PostSharpUsePipeServer` property in <xref:msbuild-properties>): on .NET Framework, and, since PostSharp 2027.0, on .NET when the project is built with the .NET SDK (`dotnet build`). The pipe server keeps the environment variables of the build that started it. When you change a variable that PostSharp reads at build time, stop the pipe servers with the `postsharp shutdown` command (`postsharp shutdown --force` for a pipe server of PostSharp 2026.0 or earlier), or disable the pipe server with `PostSharpUsePipeServer=False`, before the next build.

> [!NOTE]
> The environment variables that PostSharp reads but that this article does not list are implementation details. They may change without notice.


## Throttling Concurrent Builds

These variables limit the resources that PostSharp uses when many projects build at the same time, for instance on a build server or in a container with a memory limit. They are not set by default.

| Variable Name | Description |
|---------------|-------------|
| `POSTSHARP_MAX_PARALLELISM` | A positive integer. When this variable is set:<br>- The PostSharp MSBuild task acquires a slot from a machine-wide semaphore before it starts PostSharp. The number of slots is the value of the variable. This throttling is supported on Windows only.<br>- The number of compilations that run at the same time, in all PostSharp processes and in the pipe server, is limited to the value of the variable instead of the number of processors.<br>- PostSharp builds the type index of a target framework in one process at a time. This limits the memory used by parallel builds. |
| `POSTSHARP_CONCURRENCY_TIMEOUT_MINUTES` | A positive integer. The time, in minutes, that the PostSharp MSBuild task waits for a slot of the `POSTSHARP_MAX_PARALLELISM` semaphore. The default value is `15`. When no slot becomes available within this time, the task reports an error and the build fails. This situation happens when a build starts another build and the inner build waits for a slot that the outer build holds. <br>This variable is supported since PostSharp 2026.0.17. In earlier versions, the wait has no time limit. |
| `POSTSHARP_REQUIRED_MEMORY` | A positive integer. The amount of physical memory, in megabytes, that must be available before the PostSharp MSBuild task starts PostSharp. When less memory is available, the task waits and reports a warning. After 10 seconds, it asks the Roslyn compiler server and the Razor server to shut down (`dotnet build-server shutdown --vbcscompiler --razor`). In PostSharp 2026.0, it asks all idle .NET build servers to shut down (`dotnet build-server shutdown`). After 5 minutes, it starts PostSharp with the available memory. This variable is supported on Windows and Linux only. |


## Retrying Locked Files

When another process locks a file that PostSharp reads or writes, for instance an antivirus or another build, PostSharp retries the operation until the file is unlocked or a timeout expires. The following variables configure this behavior of the PostSharp compiler. They are supported since PostSharp 2026.0.19. The `FileLock` trace category writes the details of the retries (see the `PostSharpTrace` property in <xref:msbuild-properties>).

| Variable Name | Description |
|---------------|-------------|
| `POSTSHARP_FILE_LOCK_TIMEOUT` | A positive integer. The time, in milliseconds, after which PostSharp reports an error for a file that stays locked. The default value is `10000` (10 seconds). The first attempt always runs, even with a very short timeout. An empty or invalid value means the default value. |
| `POSTSHARP_FILE_LOCK_WARNING` | A non-negative integer. The time, in milliseconds, after which PostSharp reports a warning for a file that stays locked. By default, PostSharp reports the warning after the third failed attempt, that is, after a few tens of milliseconds. In PostSharp 2026.0, the default value is `5000` (5 seconds). The value `0` reports a warning at the first failed attempt. A value that is equal to or greater than `POSTSHARP_FILE_LOCK_TIMEOUT` disables the warning, so that only the error is reported. An operation that succeeds at the first attempt never causes a warning. |

> [!NOTE]
> The `PostSharpLockedFileTimeout` and `PostSharpLockedFileWarningTimeout` MSBuild properties do not have the same effect. They configure the copy of files that the PostSharp MSBuild targets do before and after the execution of PostSharp. See <xref:msbuild-properties>.


## Diagnostics

| Variable Name | Description |
|---------------|-------------|
| `HAS_JIT_DEBUGGER` | `True` or `False`, or `1` or `0`. Specifies whether a just-in-time debugger is installed on the machine. When `PostSharpAttachDebugger` is set to `True` and a just-in-time debugger is installed, PostSharp calls `System.Diagnostics.Debugger.Launch()`. Otherwise, PostSharp writes its process identifier to the build log and waits until you attach a debugger. By default, PostSharp assumes that a just-in-time debugger is installed on Windows, except in a container, and that none is installed on other operating systems. For details, see <xref:debugging-buildtime>. |
| `PostSharpAttachDebuggerOnException` | Any non-empty value. PostSharp calls `System.Diagnostics.Debugger.Launch()` when it reports an unhandled exception, so that you can inspect the exception in a debugger. |
| `POSTSHARPSERVER_DEBUG` | `True` to launch a debugger when the pipe server process starts. The MSBuild task then waits for the pipe server without timeout. |
| `POSTSHARP_PIPE_SERVER_LOG` | The path of a file to which the .NET pipe server writes its log: when it starts listening, and the project directory, result and duration of each compilation. `{pid}` in the path is replaced by the process identifier of the server, so that several servers write to different files. The server reads the variable when it starts, so set it in the environment of the build that starts the server, after stopping the running servers with `postsharp shutdown`. This variable is supported since PostSharp 2027.0. For details, see <xref:in-process-memory-leaks>. |


## Licensing and Usage Reporting

These variables are supported since PostSharp 2027.0.

| Variable Name | Description |
|---------------|-------------|
| `POSTSHARP_TELEMETRY_OPT_OUT` | Any non-empty value except `false` and `0` disables the reporting of usage data. For details, see <xref:whats-new-20270>. |
| `POSTSHARP_FORCE_ATTENDED` | `True` to consider the build as attended, that is, started by a developer. By default, PostSharp considers a build as unattended when it runs under a continuous integration agent, a service or a container. This affects licensing, usage reporting and notifications. It does not affect the use of the pipe server. |
| `POSTSHARP_GIT_PATH` | The path of the `git` executable. PostSharp uses `git` to determine whether a project is unmodified source code. By default, PostSharp runs `git` from the `PATH`. For details, see <xref:licensing-shared-source-code>. |


## Hosting on .NET

These variables affect the PostSharp process that runs on .NET (.NET Core), that is, when the current project does not target .NET Framework.

| Variable Name | Description |
|---------------|-------------|
| `POSTSHARPHOST_DISABLE_CACHE` | `True` to generate the *runtimeconfig.json* and *deps.json* files of the PostSharp process at every build. By default, PostSharp generates them again only when the *project.assets.json* file of the project changes. |
| `POSTSHARPHOST_DISABLE_NATIVE_FRAMEWORK_OVERRIDE` | `True` to disable the override of the native framework. When the target framework of the project has a higher version or platform version than the runtime installed on the machine, for instance `net8.0-windows10.0.22621.0`, PostSharp runs with the higher version by default, and reports a warning. With this variable, PostSharp runs with the installed version. |
| `POSTSHARPHOST_DISABLE_NATIVE_FRAMEWORK_OVERRIDE_WARNING` | `True` to remove the warning that PostSharp reports when it overrides the native framework. |
| `POSTSHARP_PIPE_SERVER_DISCRIMINATOR` | Any string. The builds of a user share one .NET pipe server for each version of PostSharp and each runtime configuration. Builds with different values of this variable use different .NET pipe servers, so that they do not share compilations, memory or environment variables, for instance two build jobs that run at the same time under the same account. This variable is supported since PostSharp 2027.0. |


## Aspect Test Framework

The aspect test framework of the `PostSharp.Testing.AspectTesting` package reads `POSTSHARP_TEST_HOLD_AT_END` and `POSTSHARP_TEST_MEMORY_LOG`, and limits the tests that weave at the same time with `POSTSHARP_MAX_PARALLELISM`. See <xref:aspect-test-reference>.


## Third-Party Environment Variables

PostSharp also reads the following environment variables, which are defined by other products.

| Variable Name | Description |
|---------------|-------------|
| `NUGET_PACKAGES` | The location of the global NuGet package folder. PostSharp searches the packages of the project in this folder. |
| `XDG_DATA_HOME`, `XDG_CONFIG_HOME` | On Linux and macOS, the base directories of the machine-wide and user NuGet configuration files. |
| `PROCESSOR_ARCHITECTURE`, `PROCESSOR_ARCHITEW6432` | On Windows, used to detect whether MSBuild runs as a 32-bit process on a 64-bit operating system. |


## See Also

**Other Resources**

<xref:msbuild-properties>
<br><xref:configuration-msbuild>
<br><xref:debugging-buildtime>
<br><xref:deploying-keys>
<br><xref:in-process-memory-leaks>
