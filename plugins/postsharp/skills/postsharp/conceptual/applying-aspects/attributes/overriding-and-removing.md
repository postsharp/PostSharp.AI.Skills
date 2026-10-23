---
uid: multicast-override
redirect_from:
  - custompatterns/aspects/applying/multicast-override
title: "Overriding and Removing Aspect Instances"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to override and remove aspect instances in PostSharp. It covers applying multiple instances of the same aspect, manually and automatically overriding an aspect instance, and deleting an aspect instance."
---
# Overriding and Removing Aspect Instances

Having multiple instances of the same aspect on the same element of code is sometimes the desired behavior. With multicasting custom attributes (<xref:PostSharp.Extensibility.MulticastAttribute>), it is easy to end up with that situation. Indeed, many multicasting paths can lead to the same target. 

However, most of the time, a different behavior is preferred. You can define a method-level aspect on the type (this aspect would apply to all methods) and override (or even exclude) the aspect on a specific method.

The multicasting engine has both the ability to apply multiple aspect instances on the same target, and the ability to replace or remove custom attributes.


## Understanding the Multicasting Algorithm

Before going ahead, it is important to understand the multicasting algorithm. The algorithm relies on a notion of *order of processing* of aspect instances. 

> [!IMPORTANT]
> This section covers how PostSharp handles multiple instances of the same aspect type for the sole purpose of computing how aspect instances should be overridden or removed. See <xref:aspect-dependencies> to understand how to cope with multiple instances of different aspects. 

The following rules apply:

* Aspect instances defined on a container (for instance a type) are always processed before instances defined on an item of that container (for instance a method). Elements of code are processed in the following order: assembly, module, type, field, property, event, method, parameter.

* When multiple aspect instances are defined on the same level, they are sorted by increasing value of the <xref:PostSharp.Extensibility.MulticastAttribute.AttributePriority>. 

The algorithm builds a list of aspect instances applied (directly and indirectly) on an element of code, sorts these instances, and processes overrides or removals as described below.


## Applying Multiple Instances of the Same Aspect

The property <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowMultiple> determines whether multiple instances of the same aspect are allowed on an element of code. By default, this property is set to `true` for all aspects. 

In the following example, the methods in type `MyClass` are enhanced by one, two and three instances of the `Trace` aspect (see code comments). 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastMultipleInstances.cs#example)]

The <xref:PostSharp.Aspects.Dependencies.AspectTypeDependencyAttribute> custom attribute specifies that several instances of the aspect can run in any order. Without it, PostSharp emits the warning <xref:ps0114?text=PS0114> because the instances are not ordered. See <xref:aspect-dependencies> for details.

When the program calls the three methods, every aspect instance writes a line:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastMultipleInstances.txt)]


## Overriding an Aspect Instance Manually

You can require an aspect instance to override any previous one by setting the aspect property <xref:PostSharp.Extensibility.MulticastAttribute.AttributeReplace>. This is equivalent to a deletion followed by an insertion (see below). 

In the following example, the first two methods of type `MyClass` are enhanced by aspects applied at assembly level, but these aspects are replaced by a different one on `Method3`. 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastReplace.cs#example)]

When the program calls the three methods, it prints:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastReplace.txt)]


## Overriding an Aspect Instance Automatically

To cause a new aspect instance to automatically override any previous one, the aspect developer must disallow multiple instances by annotating the aspect class with the custom attribute <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> and setting the property <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowMultiple> to `false`. 

In the following example, the methods in type `MyClass` are enhanced by a single `Trace` aspect: 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastAllowMultipleFalse.cs#example)]

When the program calls the three methods, it prints:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastAllowMultipleFalse.txt)]


## Deleting an Aspect Instance

The <xref:PostSharp.Extensibility.MulticastAttribute.AttributeExclude> property removes any previous instance of the same aspect on a target. 

This is useful, for instance, when you need to exclude a target from the matching set of a wildcard expression. For example:

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastExclude.cs#example)]

In this example, `ConfigurableAttribute` is a multicast attribute that is kept in the metadata (see <xref:multicast-reflection>), so the program can check by reflection to which classes it has been applied:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/MulticastExclude.txt)]
