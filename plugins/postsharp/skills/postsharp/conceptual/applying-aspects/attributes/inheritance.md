---
uid: aspect-inheritance
redirect_from:
  - custompatterns/aspects/applying/aspect-inheritance
title: "Adding Aspects to Derived Classes and Methods Using Attributes"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use PostSharp's aspect inheritance feature to apply aspects to derived classes and methods. It provides examples of applying aspects to derived types, setting inheritance on a per-usage basis, and applying aspects to new methods of derived types."
---
# Adding Aspects to Derived Classes and Methods Using Attributes

By default, aspects apply to the class or class member to which your attribute has been applied. However, PostSharp lets you specify *aspect inheritance*, which allows your attributes to be inherited by derived classes. Aspect inheritance can be specified on types, methods, and parameters, but not on properties or events. 


## Applying aspects to derived types

One way to implement aspect inheritance is to add a <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> custom attribute to your aspect class. Aspects that apply to types are typically derived from <xref:PostSharp.Aspects.TypeLevelAspect> or <xref:PostSharp.Aspects.InstanceLevelAspect>. 

The benefit of this approach is that the aspect will be automatically applied to all derived classes, eliminating the need to manually set up attributes in the derived classes. Moreover, this logic lives in one place.

The following steps describe how to enable aspect inheritance on an existing aspect that is derived from <xref:PostSharp.Aspects.TypeLevelAspect> and applies a <xref:System.Runtime.Serialization.DataContractAttribute> attribute to the base and all derived classes, and a <xref:System.Runtime.Serialization.DataMemberAttribute> attribute to all properties of the base class and those of derived classes: 


### Enabling aspect inheritance on an existing aspect

1. Create a <xref:PostSharp.Aspects.TypeLevelAspect> that implements <xref:PostSharp.Aspects.IAspectProvider>. 


2. Decorate `AutoDataContractAttribute` with the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute>, and set the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.Inheritance> to `Strict`. Note that `MulticastInheritance.Strict` and `MulticastInheritance.Multicast` have the same effect when applied to type-level aspects. 

    [!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AutoDataContract.cs?name=aspect&highlight=1)]

    > [!NOTE]
    > Aspect classes need to be serializable. For further details, see <xref:aspect-lifetime>. 


3. Decorate your base class with `AutoDataContractAttribute`. The following snippet shows a base document class and a derived article class: 

    [!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AutoDataContract.cs#classes)]


When the attribute is applied to the base class, the <xref:System.Runtime.Serialization.DataContractAttribute> and <xref:System.Runtime.Serialization.DataMemberAttribute> attributes are applied at build time to both classes. If other derived classes were added, then these would be decorated automatically as well. A program that reads these custom attributes by reflection prints:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/AutoDataContract.txt)]


## Setting inheritance on a per-usage basis

Specifying targets and attribute inheritance can also be done on a per-usage basis rather than hard-coding it into the custom attribute. In the following snippet, we have removed the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> attribute from `AutoDataContractAttribute`: 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AutoDataContractPerUsage.cs?name=aspect&highlight=1)]

Now the inheritance mode can be specified directly on the `AutoDataContractAttribute` instance by setting the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeInheritance> property as shown here: 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AutoDataContractPerUsage.cs#document)]


## Applying aspects to overridden methods

The following example shows a custom attribute that, when applied to a class, writes a message to the console window whenever a method starts and whenever it ends:

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AspectInheritanceMethod.cs#aspect)]

To specify inheritance, add the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> attribute and specify the inheritance type, or set the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeInheritance> property on the custom attribute usage. 

In the snippet below, we have added the `TraceMethod` aspect to a virtual method and used the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeInheritance> property to require the aspect to be automatically applied to all overriding methods: 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AspectInheritanceMethod.cs#classes)]

In this example, `TraceMethodAttribute` will output entry and exit messages for the `Document.RenderHtml` method and the `MultiPageArticle.RenderHtml` method, but not for the `RenderHtmlPage` method, as shown here: 

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/AspectInheritanceMethod.txt)]

> [!NOTE]
> Aspect inheritance works with virtual, abstract and interface methods and their parameters.

We would get a similar result by adding the `TraceMethod` attribute to the `Document` class. Indeed, by virtue of attribute multicasting (see section <xref:attribute-multicasting> for more details), adding a method-level attribute to a class implicitly adds it to all methods of this class. 

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AspectInheritanceClass.cs#classes)]

However, by adding the `TraceMethod` aspect to all methods of the `Document` type, we added it to the constructor and to property getters and setters, influencing the output: 

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/AspectInheritanceClass.txt)]


## Applying aspects to new methods of derived types

In the previous section the `TraceMethod` attribute used *Strict inheritance*, which means that if the base class is decorated with the attribute, the attribute is applied only to methods that are declared in the base class and overridden in the derived class. 

By changing the inheritance mode to `Multicast`, we specify that the aspect should also be applied to new methods of the derived class, that is, not only methods that are overridden from the base class. 

In the following snippet, we have changed inheritance from Strict to Multicast:

[!code-csharp[](../../../code/samples/CustomPatterns/Aspects/Applying/AspectInheritanceMulticast.cs#classes)]

With *Strict inheritance* in use, `TraceMethodAttribute` applied to `Document` was not applied to the `RenderHtmlPage` method and the `Pages` property. In other words, as the name suggests, *Strict inheritance* is strictly applying the attribute on base members and any derived members which are inherited. However, with *Multicast inheritance*, the aspect is also applied to the `RenderHtmlPage` method, the `Pages` property and the constructor of `MultiPageArticle`, as the output shows:

[!code-text[](../../../code/samples/CustomPatterns/Aspects/Applying/AspectInheritanceMulticast.txt)]

*Strict inheritance* evaluates multicasting and then inheritance, but *Multicast inheritance* evaluates inheritance and then multicasting. 

## See Also

**Reference**

<xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute>
<br><xref:PostSharp.Aspects.TypeLevelAspect>
<br><xref:PostSharp.Aspects.InstanceLevelAspect>
<br><xref:System.Runtime.Serialization.DataContractAttribute>
<br><xref:System.Runtime.Serialization.DataMemberAttribute>
<br><xref:PostSharp.Aspects.IAspectProvider>
<br><xref:PostSharp.Extensibility.MulticastAttribute>
<br><xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.Inheritance>

**Other Resources**

<xref:attribute-multicasting>
