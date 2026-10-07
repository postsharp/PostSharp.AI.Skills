---
uid: install-compiler
redirect_from:
  - deploymentconfiguration/deployment/install-compiler
title: "Installing PostSharp Into a Project"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a step-by-step guide on how to install PostSharp into a project using a PostSharp NuGet package and how to include necessary files in your source control system."
---
# Installing PostSharp Into a Project

The compiler components of PostSharp are distributed as a [PostSharp NuGet package](https://www.nuget.org/packages/PostSharp). If you want to use PostSharp in a project, you add this NuGet package to the project. 


## Adding PostSharp to a project

To add PostSharp to a project:

1. Open the **Solution Explorer** in Visual Studio. 


2. Right-click on the project.


3. Click on **Manage NuGet packages**. 


4. Click on the **Browse** tab. 


5. Search for the **PostSharp** package. 


6. Select the **PostSharp** package and click on **Install**. 


> [!TIP]
> Remember that adding PostSharp to a project means only adding the *PostSharp* NuGet package. If you want to add PostSharp to several projects in a solution, it may be easier to use NuGet to manage packages at the solution level. You may need to select the **Include Prerelease** option to install a prerelease version of PostSharp. 

> [!TIP]
> There are other ways to manage NuGet packages in your projects. See [NuGet Package Consumption Workflow documentation](https://learn.microsoft.com/en-us/nuget/consume-packages/overview-and-workflow) for more information. 

> [!TIP]
> NuGet Package Manager can be configured using a file named *nuget.config*, which can be checked into source control and can specify, among other settings, the location of the package repository (if it must be shared among several solutions, for instance) or package sources (if packages must be pre-approved). See [Common NuGet configurations](https://learn.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior) and [`nuget.config` reference](https://learn.microsoft.com/en-us/nuget/reference/nuget-config-file) for more information. 


## Including files in your source control system

After you add PostSharp to a project, you need to add the following files to source control:

* *packages.config*
* *postsharp.config*
* *\*.psproj*
* *\*.pssln*

Some of the files above might not be present depending on the package management format you use (packages.config/PackageReference) and on which PostSharp packages you have installed in your project.

Optionally, if you use the packages.config package management format, you can also include the *packages* folder in your source control. Note that there are negative consequences of this practice. See [Omitting NuGet packages in source control systems](https://learn.microsoft.com/en-us/nuget/consume-packages/packages-and-source-control) for more information. If you choose not to include the *packages* folder in your source control system, read <xref:nuget-restore>. 

Once you have all of these files included in your source code repository, any other developer getting that source code from the repository will have the required information to be able to build the application.

