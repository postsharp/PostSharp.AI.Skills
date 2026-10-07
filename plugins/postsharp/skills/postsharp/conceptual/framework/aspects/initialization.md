---
uid: aspect-initialization
redirect_from:
  - custompatterns/aspects/aspect-initialization
title: "Initializing Aspects"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to initialize aspects in PostSharp, detailing three methods: `CompileTimeInitialize`, `RuntimeInitialize`, and instance-scoped aspects methods. It also suggests when to use compile-time initialization."
---
# Initializing Aspects

As explained in the section <xref:aspect-lifetime>, a different aspect instance is associated with every element of code it is applied to. Aspect instances are created at compile time, serialized into the assembly as a managed resource, and deserialized at run time. If the aspect is instance-scoped, instances are duplicated from the prototype and initialized. 

Therefore, you can override one of the following three methods to handle aspect initializations:

* The method `CompileTimeInitialize` is invoked at compile time, and should initialize only serializable fields of the aspect, so that the value of these fields will be available at run time. The argument of this method is the <xref:System.Reflection> object representing the element of code to which this aspect instance has been applied. Therefore, this method can already perform expensive computations that depend only on metadata. 

* The method `RuntimeInitialize` is invoked at run time. Note that the aspect constructor itself is not invoked at run time. Therefore, overriding `RuntimeInitialize` is the only way to perform initialization tasks at run time. If the aspect is instance-scoped, this method is executed on the prototype instance. 

* The methods <xref:PostSharp.Aspects.IInstanceScopedAspect.CreateInstance(PostSharp.Aspects.AdviceArgs)> and <xref:PostSharp.Aspects.IInstanceScopedAspect.RuntimeInitializeInstance> are invoked only for instance-scoped aspects. They initialize the aspect instance itself, as `RuntimeInitialize` was invoked on the prototype. 

> [!TIP]
> Initializing an aspect at compile time is useful when you need to compute a difficult result that depends only on metadata, that is, it does not depend on any run-time information. An example is to build the strings that need to be printed by a tracing aspect. It is rather expensive to build strings that contain the full type name, the method name, and possibly placeholders for generic parameters and parameters. However, all required pieces of information are available at compile time. So compile time is the best moment to compute these strings.

## See Also

**Other Resources**

<xref:aspect-lifetime>
