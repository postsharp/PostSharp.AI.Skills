---
uid: contextual-dependency-resolution
redirect_from:
  - custompatterns/aspects/testing/contextual-dependency-resolution
title: "Using Contextual Dependency Resolution"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses how to use contextual dependency resolution in PostSharp, including an example of a testable logging aspect with contextual dependency resolution."
---
# Using Contextual Dependency Resolution

The dependency resolution strategy does not necessarily need to resolve to the same service implementation for all occurrences of the dependency. It is possible to design a strategy that depends on the context. For instance, the service locator could accept the aspect type and the target element of code as parameters. Test code could configure the service locator to resolve dependencies to specific implementations for a given context.

Evaluating context-sensitive rules may be CPU-intensive, but it needs to be done only during testing. In production mode, dependency resolution can be delegated to a global service catalog.


## Example: testable logging aspect with contextual dependency resolution

The following code snippet shows a logging aspect and how it could be used in production code:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/ContextualDependencyResolution.cs#production)]

The program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Testing/ContextualDependencyResolution.txt)]

The following code snippet shows how the logging aspect can be tested:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/ContextualDependencyResolutionTest.cs#test)]


