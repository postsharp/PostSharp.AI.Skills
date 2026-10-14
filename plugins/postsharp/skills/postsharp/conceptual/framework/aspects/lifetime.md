---
uid: aspect-lifetime
redirect_from:
  - custompatterns/aspects/aspect-lifetime
title: "Understanding Aspect Lifetime and Scope"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp, an Aspect-Oriented Programming (AOP) framework, instantiates aspects at compile time, unlike most other frameworks. It offers two kinds of aspect scopes: static and per-instance. The framework serializes aspect instances into a binary resource for persistence between compile time and run time."
---
# Understanding Aspect Lifetime and Scope
An original feature of PostSharp is that aspects are instantiated at compile time. Most other frameworks instantiate aspects at run time.

Persistence of aspects between compile time and run time is achieved by serializing aspect instances into a binary resource stored in the transformed assembly. Therefore, you should carefully mark all aspect classes with the <xref:PostSharp.Serialization.PSerializableAttribute> custom attribute, and distinguish between serialized fields (typically initialized at compile time and used at run time) and non-serialized fields (typically used at run time only or at compile time only). 


## Scope of Aspects

PostSharp offers two kinds of aspect scopes: static (per-class) and per-instance.


### Statically Scoped Aspects

With statically-scoped aspects, PostSharp creates one aspect instance for each element of code to which the aspect applies. The aspect instance is stored in a static field and is shared among all instances of the target class.

In generic types, the aspect instance does not have exactly the same scope as static fields. Consider the following piece of code:

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/AspectLifetimeGeneric.cs#example)]

In this program, there are two instances of the static field `f` (one for `GenericClass<int>`, the second for `GenericClass<long>`) but only a single instance of the aspect `Trace`. The aspect of this example numbers its instances, and both calls print the same number:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/AspectLifetimeGeneric.txt)]


### Instance-Scoped Aspects

Instance-scoped aspects have the same scope (instance or static) as the element of code to which they are applied. If an instance-scoped aspect is applied to a static member, it will have static scope. However, if it is applied to an instance member or to a class, it will have the same lifetime as the class instance: an aspect instance will be created whenever the class is instantiated, and the aspect instance will be garbage-collectable at the same time as the class instance.

Instance-scoped aspects are implemented according to the prototype pattern: the aspect instance created at compile time serves as a prototype, and is cloned at run time whenever the target class is instantiated. 

Instance-scoped aspects must implement the interface <xref:PostSharp.Aspects.IInstanceScopedAspect>. Any aspect may be made instance-scoped. The following code is a typical implementation of the interface <xref:PostSharp.Aspects.IInstanceScopedAspect>, taken from an aspect that counts the calls of a method for each instance of the target class: 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/AspectLifetimeInstanceScoped.cs#example)]

When the aspect is applied to a method of an `Account` class, each `Account` object gets its own aspect instance and its own counter:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/AspectLifetimeInstanceScoped.txt)]


## Steps in the Lifetime of an Aspect Instance

The following table summarizes the different steps of the aspect instance lifetime:

| Phase | Step | Description |
|-------|------|-------------|
| Compile time | Instantiation | PostSharp creates a new instance of the aspect for every target to which it applies. If the aspect has been applied using a multicast custom attribute (<xref:PostSharp.Extensibility.MulticastAttribute>), there will be one aspect instance for each matching element of code. <br>When the aspect is given as a custom attribute or a multicast custom attribute, each custom attribute instance is instantiated using the same mechanism as the Common Language Runtime (CLR) does: PostSharp calls the appropriate constructor and sets the properties and/or fields with the appropriate values. For instance, when you use the construction `[Trace(Category="FileManager")]`, PostSharp calls the default constructor and the `Category` property setter.  |
|  | Validation | PostSharp validates the aspect by calling the `CompileTimeValidate` aspect method. See <xref:aspect-validation> for details.  |
|  | Compile-time initialization | PostSharp invokes the `CompileTimeInitialize` aspect method. Concrete aspect classes may override this method to perform expensive computations that do not depend on run-time conditions. The element of code to which the aspect instance is applied is always passed to this method.  |
|  | Serialization | After the aspect instances have all been created and initialized, PostSharp serializes them into a binary stream. This stream is stored inside the new assembly as a managed resource. |
| Run time | Deserialization | Before the first aspect is executed, the aspect framework deserializes the binary stream that has been stored in a managed resource during post-compilation.<br>At this point, there is still one aspect instance per element of code to which the aspect applies. |
|  | Per-class run-time initialization | Once all custom attribute instances are deserialized, the aspect framework calls the `RuntimeInitialize` method on each of them. This time, it passes as an argument the real <xref:System.Reflection> object to which the aspect is applied.  |
|  | Per-instance run-time initialization | This step applies only to instance-scoped aspects when they have been applied to an instance member.<br>When a class is instantiated, the aspect framework creates an aspect instance by invoking the method <xref:PostSharp.Aspects.IInstanceScopedAspect.CreateInstance(PostSharp.Aspects.AdviceArgs)> of the prototype aspect instance. After the new aspect instance has been set up, the aspect framework invokes the <xref:PostSharp.Aspects.IInstanceScopedAspect.RuntimeInitializeInstance>.  |
|  | Advice Execution | Finally, advices (methods such as <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)>) are executed.  |

## See Also

**Other Resources**

<xref:aspect-validation>
<br><xref:aspect-initialization>
<br><xref:aspect-serialization>
<br><xref:instance-initialization>
