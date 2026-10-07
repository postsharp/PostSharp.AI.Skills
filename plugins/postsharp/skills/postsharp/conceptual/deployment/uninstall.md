---
uid: uninstalling
redirect_from:
  - deploymentconfiguration/deployment/uninstalling
title: "Uninstalling PostSharp"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on how to uninstall PostSharp from individual projects or solutions and from Visual Studio, including removing PostSharp packages and cleaning temporary files."
---
# Uninstalling PostSharp

If you decide to remove PostSharp from your project, we are sorry to see you leave.

There are two scenarios you may want to consider: removing PostSharp from individual projects or solutions, and removing PostSharp from Visual Studio.


## Removing PostSharp from your projects and solutions

Here are some steps to follow to remove PostSharp from your project.

> [!WARNING]
> As the following steps show, removing the product from your project is not difficult. However, replacing the aspects that you were using will be a much more arduous task that will require a great deal of planning.
> You will need to replace the aspects by handwritten source code that implements the same behaviors. Depending on how intensively you used PostSharp, your codebase could significantly increase as a result of no longer using PostSharp. Other products and frameworks that pretend to implement aspect-oriented programming actually only provide a small subset of the features you are used to with PostSharp.
> Because every project will use aspects differently, and some will have custom aspects, we are unable to provide you with any generic piece of advice about how to replace specific aspects.

To remove PostSharp from a project, you remove all PostSharp packages from it. See [Package consumption workflow](https://learn.microsoft.com/en-us/nuget/consume-packages/overview-and-workflow) for details.

Once you have removed all of the PostSharp packages from your codebase it is most probable that your application will no longer compile. Compilation errors will be registered where PostSharp aspect attributes exist in the codebase as well as where custom aspects were written. You will need to remove these entries from your codebase to get it to compile again.

Deleting the offending code can accomplish this. You must remember that in the process of removing PostSharp from your codebase these errors indicate locations where you are removing functionality from the codebase as well. If the functionality that is being removed is required by the application you will need to determine how to provide that functionality in the codebase going forward. This is the most difficult part of removing PostSharp from your codebase. Because aspects can be used in many different ways, and custom aspects can be created for any number of different uses, there is no practical way to tell you how to replace the functionality being lost.

> [!NOTE]
> You have now removed PostSharp from your codebase. At this point, you can continue your development effort without using PostSharp.
> If you would like to remove PostSharp from Visual Studio, proceed with the following steps.


## Removing PostSharp from Visual Studio

First, remove the `Metalama + PostSharp` extension from Visual Studio. See [Find, install, and manage extensions for Visual Studio](https://learn.microsoft.com/en-us/visualstudio/ide/finding-and-using-visual-studio-extensions) for details.

## Cleaning temporary files

Then you can remove the temporary files created by PostSharp. These files would be recreated as necessary the next time you run PostSharp.

Other than occupying disk space, there is no impact if you do not remove these files.

To clean up the files, delete the *PostSharp* folder from the system temporary folder. On Windows, delete the *PostSharp* folder from *%ProgramData%* as well. You can also remove the PostSharp NuGet packages from the NuGet cache, if the cache has been used to restore them. See [Managing the global packages, cache, and temp folders](https://learn.microsoft.com/en-us/nuget/consume-packages/managing-the-global-packages-and-cache-folders) for details.



