---
uid: freezable
redirect_from:
  - threading/freezable
title: "Freezable Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains the Freezable Threading Model in PostSharp, which allows users to define when an object becomes immutable. It provides steps to make an object freezable and enforce rules by the Freezable aspect."
---
# Freezable Threading Model

When you need to prevent changes to an instance of an object most of the time, but not all of the time, the Immutable pattern (implemented by the <xref:PostSharp.Patterns.Threading.ImmutableAttribute> aspect) will be too aggressive for you. In these situations, you need a pattern that allows you to define the point in time where immutability begins. To accomplish this you can make use of the <xref:PostSharp.Patterns.Threading.FreezableAttribute> aspect. 

Changes in an object with the <xref:PostSharp.Patterns.Threading.FreezableAttribute> aspect will be forbidden as soon as you call the <xref:PostSharp.Patterns.Threading.IFreezable.Freeze> method. Any further attempt to modify the object will result in an <xref:PostSharp.Patterns.Threading.ObjectReadOnlyException>. Any attempt to share the object before you call the <xref:PostSharp.Patterns.Threading.IFreezable.Freeze> method will result in a <xref:PostSharp.Patterns.Threading.ThreadMismatchException>. 

To make an object freezable, you only need to add the <xref:PostSharp.Patterns.Threading.FreezableAttribute> attribute to the class in question. 


## Making an object freezable


To make an object freezable:

1. Add the `PostSharp.Patterns.Threading` package to your project using NuGet. 


2. Add the <xref:PostSharp.Patterns.Threading.FreezableAttribute> custom attribute to your class. 

    [!code-csharp[](../../../code/samples/Threading/Freezable.cs#class)]


3. Annotate your object model for parent/child relationships as described in <xref:aggregatable-adding>. 



## Freezing an object

To freeze an object, you will first have to cast the object to the <xref:PostSharp.Patterns.Threading.IFreezable> interface. After that, you will be able to call the <xref:PostSharp.Patterns.Threading.IFreezable.Freeze> method. 

[!code-csharp[](../../../code/samples/Threading/Freezable.cs#freeze)]

> [!NOTE]
> The <xref:PostSharp.Patterns.Threading.IFreezable> interface will be injected into the `Invoice` class *after* compilation. Tools that are not aware of PostSharp may incorrectly report that the `Invoice` class does not implement the <xref:PostSharp.Patterns.Threading.IFreezable> interface. 
> Instead of using the cast operator, you can also use the <xref:PostSharp.Post.Cast``2(``0)> method. This method is faster and safer than the cast operator because it is verified and compiled by PostSharp at build time. 

> [!NOTE]
> If you are attempting to freeze either <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> or <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2> you will not be able to use the cast operator or the <xref:PostSharp.Post.Cast``2(``0)> method. Instead, you will have to use the <xref:PostSharp.Patterns.DynamicAdvising.DynamicAdvisingServices.QueryInterface``1(System.Object,System.Boolean)> method. 

Once you have called the <xref:PostSharp.Patterns.Threading.IFreezable.Freeze> method on an object instance, the code will no longer be able to change the property values on that instance. If a value change is attempted, the code will throw an <xref:PostSharp.Patterns.Threading.ObjectReadOnlyException>. 

[!code-csharp[](../../../code/samples/Threading/Freezable.cs#modify)]


## Rules enforced by the Freezable aspect

A freezable object will throw the following exceptions at run time:

* <xref:PostSharp.Patterns.Threading.ThreadMismatchException> if both of the following conditions are simultaneously true: 

    * you access the object from a different thread than the one that created it, and

    * the <xref:PostSharp.Patterns.Threading.IFreezable.Freeze> method has not yet been called. 

* <xref:PostSharp.Patterns.Threading.ObjectReadOnlyException> if a field or property is being modified after the <xref:PostSharp.Patterns.Threading.IFreezable.Freeze> method has been called. 


## Determining whether an object is in frozen state

To determine whether an object has been frozen, cast it to <xref:PostSharp.Patterns.Threading.IThreadAware> and get the read-only value from <xref:PostSharp.Patterns.Threading.IConcurrencyController.IsReadOnly> via the <xref:PostSharp.Patterns.Threading.IThreadAware.ConcurrencyController> property. 

[!code-csharp[](../../../code/samples/Threading/Freezable.cs#frozen)]


## Freezable object trees

The Freezable pattern relies on the Aggregatable pattern. The <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect will be implicitly added to the target class. Therefore, you can not only create freezable classes, but also freezable object trees. See <xref:aggregatable> for more information on how to establish object trees. 

> [!IMPORTANT]
> Children of freezable objects must be either freezable or immutable. Therefore, child classes must be annotated with the <xref:PostSharp.Patterns.Threading.FreezableAttribute> or <xref:PostSharp.Patterns.Threading.ImmutableAttribute> custom attribute. Collection types must be derived from <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> or <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2>, or be immutable collections. Arrays cannot be used. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Threading.FreezableAttribute>
<br><xref:PostSharp.Patterns.Model.AggregatableAttribute>
<br><xref:PostSharp.Patterns.Model.ChildAttribute>
<br><xref:PostSharp.Patterns.Model.ParentAttribute>
<br><xref:PostSharp.Patterns.Threading.ImmutableAttribute>
<br><xref:PostSharp.Patterns.Collections.AdvisableDictionary`2>
<br><xref:PostSharp.Patterns.Collections.AdvisableCollection`1>

**Other Resources**

<xref:immutable>
<br><xref:aggregatable>
<br><xref:aggregatable-adding>
<br><xref:advisable-collections>
