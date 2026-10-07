---
uid: license-troubleshooting
redirect_from:
  - deploymentconfiguration/licensing/license-troubleshooting
title: "License Troubleshooting"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides solutions for various technical issues that may arise with the PostSharp Licensing component, including license errors, uninstallation failures, and unexpected server disconnections."
---
# License Troubleshooting

We have designed the PostSharp Licensing component so that it interferes with your work as little as possible. However, there are legitimate cases in which we had to decide to prevent the build of your project. This article mentions these cases and gives solutions to such situations.

If the solution in this article does not work, or your case is not listed, contact our [support](https://www.postsharp.net/support). 

This article covers only the technical issues you may encounter. You can find more answers to your questions about PostSharp licensing in our [licensing FAQ](https://www.postsharp.net/il/pricing/faq). 


## Understanding typical error messages and situations

### Error PS0242: Could not find any valid registered license

If you have a license key installed as described in <xref:deploying-keys> and you still get this error, it most probably means that the version of PostSharp you are using was released after the end of your subscription period. In this case, you either need to [renew your subscription](https://www.postsharp.net/il/pricing/upgrades-and-renewals) or to use an older version of PostSharp that was released before the end of your subscription period. PostSharp reports the reason why it rejected each license key in a separate PS0301 warning. 


### Error PS0243: License error. The project uses non-licensed premium features. It is not allowed to enhance types with a total of more than X lines of code in the project by features not covered by the installed licenses, but Y lines of code were enhanced.

There are three cases in which you might get such an error:

* You are using a PostSharp Essentials license:
    In this case, you have reached the limit of the license. To diagnose this, follow the information given in [Diagnosing licensing issues](xref:express-limitations#diagnosing-licensing-issues). 

* You are using a license other than PostSharp Essentials or PostSharp Ultimate:
    If you do not have the unlimited PostSharp Ultimate license, any features not covered by your license are licensed as if you were using the PostSharp Essentials license.
    A quick way to find out which features these are is to look through the `using` directives for the `PostSharp.*` namespaces. 
    Another way is to follow the same information as for PostSharp Essentials, described above.

* You are using PostSharp Logging with a PostSharp Framework license in a version of PostSharp lower than 2025.1.
    The entitlement of a PostSharp Framework license to PostSharp Logging is hard-coded in PostSharp, not in the license key. This is a new feature of PostSharp 2025.1. If you need to use your license key with an earlier version, contact us.

### A message warning that a license or subscription has expired appears after installing a renewed license key

The expired license key is still installed. PostSharp reports its expiration in the build message `PS0304`. On Windows, PostSharp can also show a notification. In PostSharp 2026.0 and earlier, this notification was a window of the PostSharp Options application, which is no longer included in PostSharp.

Uninstall the expired license key. See <xref:deploying-keys> to find out where the expired license key can be installed. 


### A license key fails to uninstall

There are various reasons for such behavior. A quick way to resolve this is to remove the `LicenseKeys` and `LeasedLicenses` registry keys and the `LicenseKey` registry value from the following parent keys in the Windows registry: 

* `Computer\HKEY_CURRENT_USER\SOFTWARE\SharpCrafters\PostSharp 3`
* `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\SharpCrafters\PostSharp 3`

### A redistribution license does not allow building a project, although it contains only code that belongs to the licensed namespace

Besides the namespace, the name of the project must also match the licensed namespace name. Make sure that the compiled assembly name, or the name of the Git repository of the project as it appears in the URL of a remote, starts with the licensed namespace.

For example, if your licensed namespace is `MyOssLib`, your assembly name must also be `MyOssLib`, or `MyOssLib.` followed by an arbitrary string. 


### PostSharp license server is closing connections unexpectedly

Make sure that the PostSharp License Server runs on .NET Framework 4.7.2 or newer.

### The license key is valid in Visual Studio but rejected at build time

The Visual Studio extension and the build-time compiler are separate components, and they may run different versions of PostSharp. The Visual Studio extension — especially older 6.x extensions — uses its own license validation logic. As a result, a key that the Visual Studio UI accepts may still be rejected by a newer build-time compiler.

License keys are stored in the Windows registry under `HKEY_CURRENT_USER\SOFTWARE\SharpCrafters\PostSharp 3\LicenseKeys`, either directly or under a `<MinPostSharpVersion>` sub-key. This sub-key corresponds to the minimum PostSharp version the key was generated for (for example, `6.9.3`). At build time, PostSharp reads the keys stored directly under `LicenseKeys` and those under every version sub-key that is not greater than its own version. It skips a key stored under a greater version.

If the build reports PS0242 even though the key is present in the registry, use the [diagnostic logging steps](#capturing-licensing-diagnostic-logs) below to determine whether the key is being found but failing to deserialize or validate.

Common causes include:

* a license binary format incompatibility between PostSharp versions (fixed in later releases);
* a subscription that has expired relative to the PostSharp build date;
* in PostSharp 2026.0 and earlier, the pipe server caching a failed license validation from a previous build (see the next section).


### Stale pipe server and cached license failures

To improve performance, PostSharp uses a persistent pipe server process (`postsharp-x64-srv.exe`) that stays running between builds. In PostSharp 2026.0 and earlier, the pipe server caches the license state.

In these versions, once a license key is loaded and validated — or fails validation — the result is cached for the lifetime of the pipe server process. If a license key fails validation, it is permanently removed from the in-memory license collection. Subsequent builds that reuse the same pipe server will therefore see no license installed, even after you correct the underlying problem.

If you suspect stale cached state, stop the pipe server before you rebuild. Install the `postsharp` command-line tool with `dotnet tool install --global PostSharp.Tool`, then run the following command:

```powershell
postsharp shutdown --force
```

The `--force` option is required to stop a pipe server of PostSharp 2026.0 or earlier, because such a server cannot receive a request to exit. Alternatively, set `PostSharpUsePipeServer=False` to bypass the pipe server entirely.


## Capturing licensing diagnostic logs

When you need to understand why PostSharp rejects a license key, you can capture a detailed trace of the license loading and validation process.

The `PostSharpTrace` MSBuild property enables one or more trace categories. To get licensing-specific diagnostics, set it to `Licensing`. For the trace messages to appear in the build output, the build must run with the `-v:detailed` or `-v:diag` verbosity, because PostSharp emits the trace as MSBuild messages of normal importance.

> [!IMPORTANT]
> In PostSharp 2026.0 and earlier, by default on x64, PostSharp runs the license loading and validation code inside a separate AppDomain hosted by a pipe server (`PostSharpUsePipeServer=True`). In that AppDomain the trace output is silently discarded, so you will *not* see the licensing trace. To get complete licensing output, you must disable the pipe server by setting `PostSharpUsePipeServer=False`.

The following examples write the full trace to a `trace.log` file.

Using MSBuild:

```powershell
msbuild -p:PostSharpTrace="Licensing" -p:PostSharpUsePipeServer=False -t:rebuild -v:diag MyProject.csproj > trace.log 2>&1
```

Using the .NET CLI:

```powershell
dotnet build -p:PostSharpTrace="Licensing" -p:PostSharpUsePipeServer=False --no-incremental -v:diag MyProject.csproj > trace.log 2>&1
```

In the resulting log, look for the messages of the `Licensing` category. In PostSharp 2026.0 and earlier, these lines show the registry keys being loaded, the deserialization of each license, the validation results, and the requirement-satisfaction checks. Following this sequence tells you whether the license key is being found at all, and if so, whether it fails to deserialize or fails validation. From PostSharp 2027.0, these lines show whether the project is treated as unmodified, which aspects require a license that no installed key grants, and how many lines of code were enhanced under the free edition. The reason why a license key is rejected is reported as a PS0301 warning.


## Passing the license key explicitly as a workaround

If a license key is installed in the registry but PostSharp does not find it (or fails to parse it), you can bypass the registry-based resolution entirely by passing the key directly through the `PostSharpLicense` MSBuild property or a *postsharp.config* file. This is also a useful diagnostic step: it tells you whether the problem lies in reading the registry or in the license key itself.

**Option A: `Directory.Build.props` at the solution root:**

```xml
<Project>
  <PropertyGroup>
    <PostSharpLicense>YOUR-LICENSE-KEY-HERE</PostSharpLicense>
  </PropertyGroup>
</Project>
```

**Option B: `postsharp.config` at the solution root:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.postsharp.org/1.0/configuration">
  <License Value="YOUR-LICENSE-KEY-HERE" />
</Project>
```
