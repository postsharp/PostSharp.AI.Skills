---
uid: debugging-buildtime
redirect_from:
  - custompatterns/aspects/testing/debugging-buildtime
title: "Debugging Build-Time Logic"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to debug build-time logic using PostSharp, by attaching a debugger to the PostSharp process and using Visual Studio to set breakpoints."
---
# Debugging Build-Time Logic

It may seem unusual to debug build-time logic, but you can debug the build process like any other process.

To do this, attach a debugger to the PostSharp process. If you use the standard MSBuild targets for PostSharp, set the MSBuild property `PostSharpAttachDebugger` to `True`. 

This technique is easier to explain when the build-time logic (your aspect, for instance) and the transformed assembly are in different Visual Studio projects.

Suppose you have your aspect logic in `MyAspects.csproj` and unit tests (that is, the code to be transformed) in `MyAspects.Test.csproj`. To debug the build-time logic of `MyAspects.csproj`:

1. Open Visual Studio and load the solution containing `MyAspects.csproj`. 


2. Open the Visual Studio Command Prompt and go to the directory containing `MyAspects.Test.csproj`. 


3. Build `MyAspects.csproj` using Visual Studio as usual. 


4. From the command prompt, type:

    ```powershell
    msbuild MyAspects.Test.csproj /T:Rebuild /P:PostSharpAttachDebugger=True
    ```


5. The build process hits a breakpoint. When this happens, attach the Visual Studio instance that contains `MyAspects.csproj` to the build process. 

    > [!NOTE]
    > Because of a bug in Visual Studio, you need to use the **mixed debugging engine**. To do that, check the option **Manually choose the debugging engines** in the Visual Studio Just-In-Time Debugger and select both the managed and the native engines. 


6. Set breakpoints in your code and continue the program execution.



