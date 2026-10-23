---
uid: consuming-dependencies
redirect_from:
  - custompatterns/aspects/testing/consuming-dependencies
title: "Consuming Dependencies from an Aspect"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses how to consume dependencies from an aspect in PostSharp, highlighting different patterns to overcome obstacles related to instantiation and static scope."
---
# Consuming Dependencies from an Aspect

Aspects, like other components, may have dependencies on other application services. Aspects may be bound to the abstract interface of a service, and may need to resolve the dependency at run time.

However, two characteristics of aspects prevent the use of the approaches that are usual with dependency injection containers:

* Aspects are instantiated at build time, and dependency injection containers exist only at run time.

* Aspects typically have a static scope. Unless they implement <xref:PostSharp.Aspects.IInstanceScopedAspect>, aspect instances are stored in static fields, even when applied to instance members. 

These characteristics are not an obstacle to using service containers, but you must follow different patterns.

This section presents several ways to consume dependencies from an aspect:

* <xref:constructor-dependencies>: the aspect requests a service, which the target class receives from its constructors, for instance from Microsoft.Extensions.DependencyInjection. This is the approach to prefer when the objects are created by a container.
* <xref:global-service-container>
* <xref:global-service-locator>
* <xref:dynamic-dependency-resolution>
* <xref:contextual-dependency-resolution>
* <xref:importing-dependencies>

