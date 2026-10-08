---
uid: upgrade
redirect_from:
  - deploymentconfiguration/deployment/upgrade
title: "Upgrading from a Previous Version of PostSharp"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides detailed instructions on how to upgrade from a previous version of PostSharp, including upgrading Visual Studio Tools for Metalama and PostSharp and solutions from PostSharp 3 or later and PostSharp 2."
---
# Upgrading from a Previous Version of PostSharp

This section explains how to upgrade from a previous version of PostSharp.

> [!TIP]
> Other sections of this chapter, specifically <xref:installation>, <xref:deploying-keys> and <xref:build-server>, are also useful if you need to upgrade from an earlier version of PostSharp. 


## Upgrading Visual Studio Tools for Metalama and PostSharp

After you install Visual Studio Tools for Metalama and PostSharp, you will still be able to open solutions that use older versions of PostSharp.

Visual Studio Tools for Metalama and PostSharp are backward compatible with older versions of PostSharp. However, several versions of the extension cannot coexist. Therefore, installing a new version of PostSharp Tools will uninstall the previous version.

To upgrade Visual Studio Tools for Metalama and PostSharp, download it from [https://www.postsharp.net/download](https://www.postsharp.net/download) and execute the installation package. 

> [!CAUTION]
> Upgrading Visual Studio Tools for Metalama and PostSharp does not implicitly upgrade your source code.


## Upgrading solutions from PostSharp 3 or later

> [!CAUTION]
> Before you upgrade your project to a different major release of PostSharp, check that the new version still supports your version of Visual Studio and the target framework of your application. Check the release notes for an accurate compatibility list of the specific version you are installing.

You can use several versions of PostSharp side-by-side on the same machine. However, it is recommended that you use the same version in all projects of the same solution. Within a project, the *PostSharp* and *PostSharp.Redist* packages must have the same version. Otherwise, the build fails or emits a warning. See <xref:postsharp-components> for details.


To upgrade a solution from PostSharp 3 or later:

1. Open the **Solution Explorer** in Visual Studio. 


2. Right-click on the solution.


3. Click on **Manage NuGet Packages for Solution**. 


4. Click on **Updates**. 


5. Find the *PostSharp* package and click on **Update**. 


6. Select all projects and click **OK**. 


7. Repeat the operation for all *PostSharp.Patterns.** packages. 



## Upgrading large repositories

If your source contains a large number of solutions, upgrading manually using NuGet may be too labor-intensive. In this situation, it is better to use our upgrade PowerShell script.

To upgrade a large number of solutions with the PowerShell script:

1. Download the following Git repository: [https://github.com/sharpcrafters/PostSharp.Utilities](https://github.com/sharpcrafters/PostSharp.Utilities). You can download it manually from the web page or execute the following command: 

    ```none
    git clone https://github.com/sharpcrafters/PostSharp.Utilities.git
    ```


2. Follow the instructions in *README.md*. 


> [!WARNING]
> This script does not support platforms other than the .NET Framework and does not support PostSharp Pattern Libraries.


## Upgrading solutions from PostSharp 2

Every project can have references to only a single version of PostSharp. This applies both to direct and indirect references. The PostSharp 3 or later compiler is not backward compatible with PostSharp 2, and PostSharp 3 will refuse to compile projects that have a reference to PostSharp 2. Therefore, you will typically use a single version of PostSharp in every solution.

You can upgrade projects from PostSharp 2 to PostSharp 3 by adding the *PostSharp* NuGet package to these projects. 


To upgrade a solution from PostSharp 2:

1. Open the **Solution Explorer** in Visual Studio. 


2. Right-click on the solution.


3. Click on **Manage NuGet Packages for Solution**. 


4. Click on **Online**. 


5. In the search box, type `PostSharp`. You may want to select the **Select prereleases** option (instead of the default **Stable Only**) to install a pre-release version of PostSharp. 


6. Find the *PostSharp* package and click on **Install**. 


7. Select all projects and click **OK**. 


Although PostSharp 3 or later is mostly backward compatible with PostSharp 2 at source-code level, you may need to perform small adjustments to your source code:

* Every occurrence of the <xref:System.Runtime.InteropServices._Assembly> interface has been replaced by the <xref:System.Reflection.Assembly> class. You may have to change the signatures of some methods derived from <xref:PostSharp.Aspects.AssemblyLevelAspect>. 

* Aspects that target Silverlight, Windows Phone or Windows Store must be annotated with the <xref:PostSharp.Serialization.PSerializableAttribute> custom attribute. 

* PostSharp Toolkits 2.1 need to be uninstalled using NuGet. Instead, you can install PostSharp Pattern Libraries 3 from NuGet. Namespaces and some type names have been changed.


