---
uid: thread-safety-policy
redirect_from:
  - threading/thread-safety-policy
title: "Making a Whole Project or Solution Thread Safe"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions on how to implement thread-safety policies in a project or solution using PostSharp threading models, either manually or via Visual Studio Tools for Metalama and PostSharp."
---
# Making a Whole Project or Solution Thread Safe

When you want to make a large application thread-safe with PostSharp threading models, it can become difficult to remember to assign a threading model to every single class. In this situation, you can add the thread-safety policy to your project or solution.

The thread-safety policy emits warnings in two situations:

* classes that are not assigned to a threading model,

* static fields that are not read-only or not of a thread-safe type.

For instance, when the policy is enabled, the following code produces two warnings:

[!code-csharp[](../../../code/samples/Threading/Core/ThreadSafetyPolicyWarnings.cs#example)]

[!code-text[](../../../code/samples/Threading/Core/ThreadSafetyPolicyWarnings.build.txt)]

> [!IMPORTANT]
> The thread-safety policy does not make your application thread-safe by itself. What the thread-safety policy does is to remind you to use threading models in your code. It is the use of threading models that makes your application thread-safe.


## Adding the thread-safety policy using Visual Studio Tools for Metalama and PostSharp


To apply the thread-safety policy to your application with Visual Studio Tools for Metalama and PostSharp:

1. Right-click your solution or your project in **Solution Explorer**, select **Add** followed by **PostSharp Policy...**

    ![](thread-safety-policy-1.png)


2. In the **Add PostSharp policy** wizard, expand **Threading** and select **Thread Safety**. 


3. If you selected the solution, select the projects that you would like to add the policy to.


4. Review the configuration that you have selected and click **Next**. 


5. Close the wizard when the process has completed by clicking **Finish**. 


If you added the policy to the whole solution, the result of running this wizard is that a `pssln` file has been added to your project. The `pssln` file contains an entry that enables the thread-safety policy across all projects in your solution. 

```xml
<Project xmlns="http://schemas.postsharp.org/1.0/configuration" xmlns:t="clr-namespace:PostSharp.Patterns.Threading;assembly:PostSharp.Patterns.Threading">
  <Multicast>
    <t:ThreadSafetyPolicy />
  </Multicast>
</Project>
```


## Adding the thread-safety policy to a project manually


To add the thread-safety policy to a project manually:

1. Add the `PostSharp.Patterns.Threading` NuGet package to the project. 


2. Add the <xref:PostSharp.Patterns.Threading.ThreadSafetyPolicy> to any C# file. We recommend you add it to a new file named `GlobalAspects.cs`. 

    [!code-csharp[](../../../code/samples/Threading/Core/ThreadSafetyPolicyWarnings.cs#policy)]



## Adding the thread-safety policy to a whole solution manually


To manually add the thread-safety policy to a whole solution:

1. Open the solution's `pssln` file. This can be found under the Solution Items folder in Visual Studio's Solution Explorer.

    ![](thread-safety-policy-2.png)

    If the `pssln` file does not exist, add the file manually at the solution level. Name the file with the same name as your solution and the `pssln` file extension. 


2. If you had to create the `pssln` file and add it to your solution, add the following XML to it. If the `pssln` file already existed in your project, proceed to the next step. 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration" xmlns:t="clr-namespace:PostSharp.Patterns.Threading;assembly:PostSharp.Patterns.Threading">
    </Project>
    ```


3. Add a multicast attribute to the Project element that will add <xref:PostSharp.Patterns.Threading.ThreadSafetyPolicy> to all the projects in the solution. 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration" xmlns:t="clr-namespace:PostSharp.Patterns.Threading;assembly:PostSharp.Patterns.Threading">
      <Multicast>
        <t:ThreadSafetyPolicy />
      </Multicast>
    </Project>
    ```


4. Add the `PostSharp.Patterns.Threading` NuGet package to all projects in the solution. 


Once you save the `pssln` file, you will have added the thread-safety policy to all projects in your solution. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Threading.ThreadSafetyPolicy>
