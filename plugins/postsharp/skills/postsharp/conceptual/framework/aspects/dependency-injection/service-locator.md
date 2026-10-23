---
uid: global-service-locator
redirect_from:
  - custompatterns/aspects/testing/global-service-locator
title: "Using a Global Service Locator"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides guidance on using a global service locator in PostSharp for dependency injection, to avoid wasting memory. It includes example code for a logging aspect."
---
# Using a Global Service Locator

If all aspect instances use the same global dependency injection container, the dependencies of all instances are likely to resolve to the same service implementation. Therefore, storing dependencies in an instance field may waste memory, especially for aspects that are applied to a very high number of code elements.

Alternatively, you can store dependencies in static fields and initialize them in the static constructor of the aspect.

> [!TIP]
> Use the <xref:PostSharp.Extensibility.PostSharpEnvironment.IsPostSharpRunning> property to make sure that this part of the static constructor is executed at run time only, when PostSharp is *not* running. 

In this case, you cannot use dependency injection methods such as <xref:System.ComponentModel.Composition.Hosting.CompositionContainer.SatisfyImportsOnce(System.ComponentModel.Composition.Primitives.ComposablePart)>. Instead, you must use the container as a service locator. For instance, MEF exposes the method <xref:System.ComponentModel.Composition.Hosting.ExportProvider.GetExport*>. 

> [!IMPORTANT]
> The service locator must be initialized before the execution of any class that is enhanced by the aspect. Therefore, you cannot use the aspect on the entry-point class (typically `Program` or `App`). To relax this constraint, you can initialize the dependency on demand, for instance by using <xref:System.Lazy`1>. 


## Example: testable aspect with a global MEF service locator

The following code snippet shows a logging aspect and how it could be used in production code:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/GlobalServiceLocator.cs#production)]

The program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Testing/GlobalServiceLocator.txt)]

The following code snippet shows how the logging aspect can be tested:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/GlobalServiceLocatorTest.cs#test)]


