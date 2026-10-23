---
uid: global-service-container
redirect_from:
  - custompatterns/aspects/testing/global-service-container
title: "Using a Global Composition Container"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to initialize an aspect from an ambient container at runtime using a global composition container in the PostSharp AOP framework. It includes relevant code examples."
---
# Using a Global Composition Container

The dependency injection container cannot instantiate the aspect, but you can initialize the aspect from an *ambient container* at run time. An ambient container is exposed as a static member and is global to the whole application. 

Dependency injection containers typically offer methods to initialize objects that have been instantiated externally. For instance, the Managed Extensibility Framework offers the <xref:System.ComponentModel.Composition.Hosting.CompositionContainer.SatisfyImportsOnce(System.ComponentModel.Composition.Primitives.ComposablePart)> method. 

The dependency injection method can be invoked from the <xref:PostSharp.Aspects.MethodLevelAspect.RuntimeInitialize(System.Reflection.MethodBase)> method. 

> [!NOTE]
> User code has no control over when and on which thread an aspect is initialized. Therefore, using <xref:System.ThreadStaticAttribute> to make the container local to the current thread is not a reliable approach. 

> [!IMPORTANT]
> The service container must be initialized before the execution of any class that is enhanced by the aspect. Therefore, you cannot use the aspect on test classes themselves. To relax this constraint, you can initialize the dependency lazily, when the first advice is executed.


## Example: testable logging aspect with a global MEF service container

The following code snippet shows a logging aspect and how it could be used in production code:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/GlobalServiceContainer.cs#production)]

The program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Testing/GlobalServiceContainer.txt)]

The following code snippet shows how the logging aspect can be tested:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/GlobalServiceContainerTest.cs#test)]


