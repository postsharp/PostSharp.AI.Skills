---
uid: instance-initialization
redirect_from:
  - custompatterns/aspects/advanced/instance-initialization
title: "Coping with Custom Object Serializers"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to handle custom object serializers in PostSharp. It details how to manually initialize aspects either by defining a method 'InitializeAspects' or invoking 'AspectUtilities.InitializeCurrentAspects'."
---
# Coping with Custom Object Serializers

Some aspects need to be initialized when a new instance of the class to which they are applied is created. For instance, instance-scoped aspects must be cloned from the prototype, and members imported through <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> must be bound to aspect fields. 

PostSharp enhances every constructor of every enhanced class so that aspects are properly initialized.

However, it is possible to create new instances of classes by *bypassing* the constructor. This happens, for instance, when classes are deserialized by the <xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter> or the <xref:System.Runtime.Serialization.DataContractSerializer>. These formatters use the method <xref:System.Runtime.Serialization.FormatterServices.GetUninitializedObject(System.Type)> to create new instances, but this method bypasses all constructors. 

PostSharp implements a workaround for the deserializers <xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter> and <xref:System.Runtime.Serialization.DataContractSerializer>: it creates or modifies a method annotated by the custom attribute <xref:System.Runtime.Serialization.OnDeserializingAttribute>, so that aspects are initialized properly. 

However, if you are using a custom deserializer, or for any reason create instances using the method <xref:System.Runtime.Serialization.FormatterServices.GetUninitializedObject(System.Type)>, you must initialize aspects manually. 


## Initializing Aspects Manually

There are two ways to initialize an aspect from user code.


### By Defining a Method InitializeAspects

You can define in your classes (typically in one of the root classes of your class hierarchy) a method with the following name and signature:
<!-- not-compiled: signature only -->

```csharp
protected virtual void InitializeAspects();
```

When PostSharp discovers this method, it adds its own initialization logic at the end of the `InitializeAspects` method, after the original logic. The original logic is not deleted. This method can safely have an empty implementation. 

The following constraints apply:

* The method should be `virtual` unless the class is sealed. 

* The method should be `protected` or `public` unless the class is `internal`. 

For instance, the following class would enable aspects (applied to this class or on derived classes) to be initialized after deserialization (PostSharp automatically generates this code for <xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter> and <xref:System.Runtime.Serialization.DataContractSerializer>, so you need to write it manually only for a custom serializer). 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Advanced/InstanceInitialization.cs#example)]

A method annotated with <xref:System.Runtime.Serialization.OnDeserializingAttribute> must have a parameter of type <xref:System.Runtime.Serialization.StreamingContext>.

In the following program, a `Customer` class derived from `BaseClass` has a method enhanced by an instance-scoped aspect that counts the calls. After a `Customer` object is deserialized by <xref:System.Runtime.Serialization.DataContractSerializer>, the aspect works as usual:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Advanced/InstanceInitialization.txt)]


### By Invoking AspectUtilities.InitializeCurrentAspects

Instead of providing an empty method `InitializeAspects`, you can invoke the method <xref:PostSharp.Aspects.AspectUtilities.InitializeCurrentAspects>. PostSharp translates a call to this method into a call to `InitializeAspects`. You must invoke it from a non-static method of an enhanced class. 

If the class from which <xref:PostSharp.Aspects.AspectUtilities.InitializeCurrentAspects> is invoked has not been enhanced by an aspect requiring initialization, the call to this method is simply ignored. 

> [!NOTE]
> Using this approach may be brittle in some situations: calls to <xref:PostSharp.Aspects.AspectUtilities.InitializeCurrentAspects> have no effect if aspects are applied to derived classes, but not to the calling class. In this scenario, it is preferable to define the method `InitializeAspects`. 


