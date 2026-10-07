---
uid: configuration-postsharp
redirect_from:
  - deploymentconfiguration/configuration/configuration-postsharp
title: "Well-Known PostSharp Properties"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a list of well-known PostSharp properties that can be set from the MSBuild project, their corresponding MSBuild property names, and descriptions. It also mentions the option of defining custom PostSharp properties."
---
# Well-Known PostSharp Properties

The following table lists the PostSharp properties that may be set from the MSBuild project. The second column specifies the name of the MSBuild property that influences the value of the PostSharp property, if any.

| Property name | MSBuild property name | Description |
|---------------|-----------------------|-------------|
| `Configuration` | `Configuration` | Build configuration (typically `Debug` or `Release`).  |
| `Platform` | `Platform` | Target processor architecture (typically `AnyCPU`, `x86` or `x64`).  |
| `MSBuildProjectFullPath` | `MSBuildProjectFullPath` | Full path of the C# or VB project being built. |
| `IgnoredAssemblies` |  | Comma-separated list of assembly short names (without extension) that should be ignored by the dependency scanning algorithm. Add an assembly to this list if it is obfuscated, or contains native code, and causes PostSharp to fail. |
| `ReferenceDirectory` | `MSBuildProjectDirectory` | Directory with respect to which relative paths are resolved. |
| `SearchPath` | `PostSharpSearchPath` | Semicolon-separated list of directories containing reference assemblies and plug-ins. |
| `TargetFrameworkMoniker` | `TargetFrameworkIdentifier`, `TargetFrameworkVersion` | Moniker of the target framework of the current project (that is, the framework on which the application will run). For example, `.NETFramework,Version=v4.7.2`.  |

Other properties are recognized but are of little interest to end users. For a complete list of properties, see *PostSharp.targets*. 


## Using custom properties

By defining your own PostSharp properties, you can pass information from the build environment to aspects, or to any code running in PostSharp. Custom PostSharp properties behave exactly like other PostSharp properties, so they can be defined and read using the same procedures.


