---
uid: dynamic-dependency-resolution
redirect_from:
  - custompatterns/aspects/testing/dynamic-dependency-resolution
title: "Using Dynamic Dependency Resolution"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses dynamic dependency resolution in PostSharp, which allows different configurations for testing and production. It presents an example of a testable logging aspect with a global MEF service container."
---
# Using Dynamic Dependency Resolution

Both previous approaches have a static dependency resolution strategy: it cannot be changed over time. Therefore, these strategies may be unsuitable when several tests need different configurations of the dependency container.

A possible solution is to resolve dependencies dynamically each time they are needed, and not only at aspect initialization. This solution is ideal for testing, but it may be too inefficient for production. Therefore, the solution must still provide dependency caching for production mode. Caching neutralizes the dynamic characteristics of dependency resolution.

This solution is based on the following elements:

* The service locator can be initialized in two modes: production (the resolution strategy is immutable) and testing (the resolution strategy can be modified).

* The service locator returns a delegate (`Func<T>`, where *T* is the dependency type), instead of the dependency itself (`T` or `Lazy<T>`). 

* The aspect calls the service locator during aspect initialization and stores the delegate.

* The aspect calls the delegate at run time.


## Example: testable logging aspect with a global MEF service container with dynamic resolution

The following code snippet shows a logging aspect and how it could be used in production code:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/DynamicDependencyResolution.cs#production)]

The program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Testing/DynamicDependencyResolution.txt)]

The following code snippet shows how the logging aspect can be tested:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/DynamicDependencyResolutionTest.cs#test)]


