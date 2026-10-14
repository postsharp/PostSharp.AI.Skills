---
uid: attribute-introduction
redirect_from:
  - custompatterns/aspects/tutorials/attribute-introduction
title: "Introducing Custom Attributes"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document introduces PostSharp's ability to create custom attribute classes in C#, which can automatically decorate class members with custom attributes. It also explains how to copy attributes from one class to another."
---
# Introducing Custom Attributes

Applying custom attributes to class members in C# is a powerful way to add metadata about those members at compile time.

PostSharp provides the ability to create a custom attribute class that, when applied to another class, can iterate through those class members and automatically decorate them with custom attributes. This can be useful, for example, to automatically apply custom attributes or groups of custom attributes when new class members are added, without having to remember to do it manually each time.


## Introducing new custom attributes

In the following example, we create an attribute decorator class that applies the .NET <xref:System.Runtime.Serialization.DataContractAttribute> to a class and <xref:System.Runtime.Serialization.DataMemberAttribute> to members of a class at build time. 


1. Start by creating a class called `AutoDataContractAttribute` that derives from <xref:PostSharp.Aspects.TypeLevelAspect>. <xref:PostSharp.Aspects.TypeLevelAspect> transforms the class into an attribute that can be applied to other classes. Also implement <xref:PostSharp.Aspects.IAspectProvider>, which exposes the <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> method for iterating on class members. <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> is called once for each class to which the aspect is applied. Like all aspects, the class must be annotated with <xref:PostSharp.Serialization.PSerializableAttribute>.

2. Implement the <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> method to cast the `targetElement` parameter to a `Type` object. Note that this method will be called at build time. The `Type` object can be used for inspecting each member and making decisions about when and how to apply custom attributes. In the following snippet, the implementation returns a new <xref:PostSharp.Aspects.AspectInstance> for the `Type` containing a new <xref:System.Runtime.Serialization.DataContractAttribute> and then iterates through each property of the `Type` returning a new <xref:PostSharp.Aspects.AspectInstance> with the <xref:System.Runtime.Serialization.DataMemberAttribute> for each. Note that both the <xref:System.Runtime.Serialization.DataContractAttribute> and <xref:System.Runtime.Serialization.DataMemberAttribute> are both wrapped in <xref:PostSharp.Aspects.CustomAttributeIntroductionAspect> objects: 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/AutoDataContract.cs#aspect)]

    > [!NOTE]
    > Since the `ProvideAspects` method returns an `IEnumerable`, use the `yield` keyword to return aspects for PostSharp to apply. 


3. Apply the `AutoDataContractAttribute` class. In the following example, we apply it to a `Product` class where it will decorate `Product` with <xref:System.Runtime.Serialization.DataContractAttribute> and each member with <xref:System.Runtime.Serialization.DataMemberAttribute>: 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/AutoDataContract.cs#product)]

    When you inspect the `Product` class with reflection after the build, you see the following attributes:

    [!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/AutoDataContract.txt)]



## Copying existing custom attributes

Another way to introduce attributes to class members is to copy them from another class. This is useful, for example, when distinct classes have members with the same names and the same types. In this case, attributes can be defined in one class and then that class can be used to decorate other similar classes with the same attributes.

In the following snippet, the `Id` and `Name` properties of `Product` have both been modified to contain attributes from the `System.ComponentModel.DataAnnotations` namespace: `Editable` and `Required` on `Id`, and `Display` and `Required` on `Name`. We want to copy these attributes to another class called `ProductViewModel`, which contains the same properties: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/CopyCustomAttributes.cs#product)]

To copy the attributes from the properties of `Product` to the corresponding properties of `ProductViewModel`, create an attribute class which can be applied to `ProductViewModel` to perform this copy process: 


1. Create a <xref:PostSharp.Aspects.TypeLevelAspect> that implements <xref:PostSharp.Aspects.IAspectProvider>. In the following snippet, the class is called `CopyCustomAttributesFrom`. Its constructor takes the type from which the property attributes are copied. Then implement <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)>: 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/CopyCustomAttributes.cs#aspect)]

    The <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> method iterates through each property of the target class and then gets the corresponding property from the source class. It then iterates through all custom attributes defined for the source property, copying each to the corresponding property of the target class. <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> also filters out attributes that are not from the `System.ComponentModel.DataAnnotations` namespace to demonstrate how you may want to ignore some attributes during the copy process. 


2. Decorate the `ProductViewModel` class with the `CopyCustomAttributesFrom` attribute, specifying `Product` as the source type in the constructor. During compilation, the <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> method of `CopyCustomAttributesFrom` will then perform the copy process from `Product` to `ProductViewModel`: 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/CopyCustomAttributes.cs#view-model)]


The following program prints the data annotations of the properties of `ProductViewModel` using reflection:

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/CopyCustomAttributes.cs#program)]

The output shows that the `Editable`, `Display` and `Required` attributes were copied from `Product` to `ProductViewModel` at build time:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/CopyCustomAttributes.txt)]

> [!NOTE]
> It is not possible to delete or replace an existing custom attribute.

## See Also

**Reference**

<xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)>
<br><xref:PostSharp.Aspects.Advices.CopyCustomAttributesAttribute>
<br><xref:System.Runtime.Serialization.DataContractAttribute>
<br><xref:System.Runtime.Serialization.DataMemberAttribute>
<br><xref:PostSharp.Aspects.TypeLevelAspect>
<br><xref:PostSharp.Aspects.IAspectProvider>
<br><xref:PostSharp.Aspects.AspectInstance>
<br><xref:PostSharp.Aspects.CustomAttributeIntroductionAspect>
