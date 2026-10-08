---
uid: simple-aspects
redirect_from:
  - custompatterns/aspects/simple-aspects
title: "Developing Simple Aspects"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on developing simple aspects in PostSharp, which are composed of a single transformation, and includes related articles for further reading."
---
# Developing Simple Aspects

Simple aspects are aspects that are composed of a single transformation. Developing a simple aspect in PostSharp is straightforward: you create a new class, derive it from a primitive aspect class, and override some special methods named *advices*. 

If your aspect cannot be implemented as a single transformation, see <xref:member-level-aspects> for several behaviors on one member, or <xref:complex-aspects> for behaviors on several members of a type or an assembly. 

The methods of a simple aspect receive an object, such as <xref:PostSharp.Aspects.MethodExecutionArgs>, that is allocated on each call and gives untyped access to the arguments and the return value. To avoid this allocation and get typed values, write the aspect with advices and bound parameters instead (see <xref:modernizing-aspects>). 


## In this section

| Article | Description |
|---------------------------------------------|-------------------------------------------------|
| <xref:method-decorator> | This article shows how to execute code when a method starts, succeeds, or fails, around `await` operators of `async` methods, or `yield return` statements of iterator methods.  |
| <xref:exception-handling> | This article describes how to write an aspect that handles exceptions. |
| <xref:method-interception> | This article explains how to intercept the execution of methods. |
| <xref:location-interception> | This article shows how to intercept read and write accesses to fields and properties. |
| <xref:event-interception> | This article describes how to intercept the action of adding a delegate to an event, removing a delegate from an event, and invoking a delegate by raising the event. |
| <xref:attribute-introduction> | This article shows how to add a custom attribute to any element of code. |
| <xref:resource-introduction> | This article shows how to add a managed resource to the current assembly. |


