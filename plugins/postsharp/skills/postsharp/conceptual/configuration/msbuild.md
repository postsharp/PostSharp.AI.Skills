---
uid: configuration-msbuild
redirect_from:
  - deploymentconfiguration/configuration/configuration-msbuild
title: "Configuring Projects Using MSBuild"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides guidance on how to configure PostSharp projects using MSBuild properties, including setting properties with a text editor, environment variables, and from the command line."
---
# Configuring Projects Using MSBuild

Most configuration settings of PostSharp can be set as MSBuild properties. This topic describes how to modify them.


## List of well-known properties

See <xref:msbuild-properties>. 


## Setting MSBuild properties with a text editor

To set a property that persistently applies to a specific project, but not to the whole solution, the best solution is to define it directly inside the C# or VB project file (`*.csproj` or `*.vbproj`, respectively) using a text editor. 

1. Open the **Solution Explorer**, right-click on the project name, click on **Unload project**, then right-click again on the same project and click on **Edit**. 


2. Insert the following XML fragment just before the `<Import />` elements: 

    ```xml
    <PropertyGroup>
        <PostSharpEscalatedMessages>*</PostSharpEscalatedMessages>
    </PropertyGroup>
    ```


3. Save the file. If the project was open in Visual Studio, go to the Solution Explorer, right-click on the project name, then click on **Reload project**. 



## Setting MSBuild properties with environment variables

Since all environment variables are imported into MSBuild properties, you can set any MSBuild property by setting the environment variable of the same name before starting the MSBuild process. This is a convenient way to set up PostSharp from a build server.


## Configuring several projects at a time

Instead of editing every project file, you can define shared settings in a file named *Directory.Build.props* and stored in the same directory as the project file or in any parent directory of the project file. 

With this mechanism, you can define settings that apply to a large set of projects and control the granularity of settings.

Files *Directory.Build.props* are ordinary MSBuild project or targets files. They should have the following content: 

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <PostSharpLicense>XXX-AAAAA</PostSharpLicense>
  </PropertyGroup>
</Project>
```

> [!NOTE]
> Before MSBuild added support for *Directory.Build.props*, PostSharp implemented the same feature using a file named *PostSharp.Custom.targets*, which was searched for in up to 8 levels of parent directories. This feature is still supported, but we recommend using *Directory.Build.props*. 


## Setting MSBuild properties from the command line

When an MSBuild property does not need to be set permanently, it is convenient to set it from the command prompt by appending the flag `/p:PropertyName=PropertyValue` to the command line of `msbuild.exe`. For example: 

```powershell
msbuild.exe /p:PostSharpUsePipeServer=False
```

## See Also

**Other Resources**

[MSBuild Properties](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-properties)
<br>[Directory.Build.props and Directory.Build.targets](https://docs.microsoft.com/en-us/visualstudio/msbuild/customize-your-build)
