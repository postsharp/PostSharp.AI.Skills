---
uid: install-without-nuget
redirect_from:
  - deploymentconfiguration/deployment/install-without-nuget
title: "Installing PostSharp without NuGet (Deprecated)"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions for installing PostSharp without using NuGet, an unsupported method primarily intended for users of legacy .NET Framework projects."
---
# Installing PostSharp without NuGet

> [!WARNING]
> This section describes a way of consuming PostSharp that is unsupported and is provided for the convenience of users of legacy (non-SDK) .NET Framework projects. 
> You should not target .NET Standard, .NET Core or .NET while using PostSharp from the ZIP archive.

The most common way to add PostSharp to your project is by installing PostSharp NuGet packages. The main benefit of using NuGet Package Manager is that it provides a standard way to install and manage all dependencies for your .NET projects.

Previous versions of NuGet had several issues that made it an impractical solution for some teams. For this reason, we allow using PostSharp without NuGet, by downloading and extracting a standard zip file. However, the installation procedure is significantly more cumbersome without NuGet than with NuGet.

## Installing PostSharp into a project without NuGet

1. Download the zip distribution from [https://www.postsharp.net/downloads](https://www.postsharp.net/downloads) (a file named PostSharp-x.x.x.zip). 


2. Extract the zip file into a local directory.


3. Using Visual Studio, add references to the relevant PostSharp assemblies. They are located under the *lib\net472* directory. 


4. Open the project file with a text editor and add the following line after the last `Import` element. 

    ```xml
    <Import Project="..\..\..\postsharp\Tools\PostSharp.targets" />
    ```

## See Also

**Other Resources**

<xref:deployment>
<br><xref:install-compiler>
<br><xref:uninstalling>
