---
uid: location-interception
redirect_from:
  - custompatterns/aspects/tutorials/location-interception
title: "Intercepting Properties and Fields"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "This document provides a detailed guide on how to intercept properties and fields in .NET using PostSharp's LocationInterceptionAspect. It covers intercepting get and set operations, getting and setting the underlying property, intercepting fields, and reacting to initialization of instance fields and properties."
---
# Intercepting Properties and Fields

In .NET, both fields and properties are elements that can be read and written. You can intercept get and set operations using the <xref:PostSharp.Aspects.LocationInterceptionAspect>. It makes it possible to develop useful aspects, such as validation, filtering, change tracking, change notification, or property virtualization (where the property is backed by a registry value, for instance). 


## Intercepting get operations

In this example, we will see how to create an aspect that filters the value read from a field or property.


### Creating an aspect that filters the value read from a field or property

1. Add a reference to the *PostSharp* package to your project. 


2. Create an aspect that inherits from <xref:PostSharp.Aspects.LocationInterceptionAspect> and add the custom attribute [<xref:PostSharp.Serialization.PSerializableAttribute>]. 


3. Override the <xref:PostSharp.Aspects.LocationInterceptionAspect.OnGetValue(PostSharp.Aspects.LocationInterceptionArgs)> method. Calling `base.OnGetValue` actually retrieves the value from the underlying field or property, and populates the `Value` property. Add some code to check if the property currently is set to `null`. If the current value is `null`, we want to return a predefined value. To do this we can set the `Value` property. Any time this property is requested, and it is set to `null`, the value `"foo"` will be returned. 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptGet.cs#aspect)]


4. Now that you have a complete getter interception aspect, you can attach it to the target code. Add the attribute to properties or fields to intercept them.

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptGet.cs#customer)]

    > [!NOTE]
    > Adding aspects to target code one property or field at a time can be a tedious process. There are a number of techniques in the article <xref:attribute-multicasting> that explain how to add aspects en masse. 


5. Now when you create an instance of a customer and immediately try to access the `Name` and `Address` values, the get request will be intercepted and null values will be returned as `"foo"`. 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptGet.cs#program)]

    The program prints the following output:

    [!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptGet.txt)]


Property and field interception is a simple and seamless task. Once you have intercepted your target, you can act on the target or allow the original code to execute.


## Intercepting set operations

The previous section showed how to intercept a get accessor. Intercepting a set accessor is accomplished in a similar manner by implementing <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> in the <xref:PostSharp.Aspects.LocationInterceptionAspect>. 


When applied to a property with a set accessor, <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> will intercept the set operation. In the `Customer` example shown below, <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> will be called whenever the `Name` property is set: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptSet.cs#customer)]

You can set the `Value` property of <xref:PostSharp.Aspects.LocationInterceptionArgs> and then call <xref:PostSharp.Aspects.LocationInterceptionArgs.ProceedSetValue> to pass a different value for the property. For example, <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> could be used to check for a null string, and then change the string to a non-null value: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptSet.cs#aspect)]

When the program sets the `Name` property to `"Jane"` and then to `null`, it prints the following values of the property:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptSet.txt)]

> [!WARNING]
> Inline initializers of instance fields and properties are not intercepted by <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)>. To react to the initialization of an instance field or property, use the <xref:PostSharp.Aspects.LocationInterceptionAspect.OnInstanceLocationInitialized(PostSharp.Aspects.LocationInitializationArgs)> advice. See [Reacting to initialization of instance fields and properties](#reacting-to-initialization-of-instance-fields-and-properties) for details. 


## Getting and setting the underlying property

PostSharp provides a mechanism to check a property's underlying value via the <xref:PostSharp.Aspects.LocationInterceptionArgs.GetCurrentValue> method. This can be useful to check the current property value when a setter is called and then take some appropriate action. 

For example, the following snippet shows a modified <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> method that gets the current underlying property value and compares the (new) value passed into the setter against the current value. If the current and new values do not match, a message is written: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/UnderlyingValue.cs#on-set-value)]

> [!NOTE]
> <xref:PostSharp.Aspects.LocationInterceptionArgs.GetCurrentValue> will call the underlying property getter without going through <xref:PostSharp.Aspects.LocationInterceptionAspect.OnGetValue(PostSharp.Aspects.LocationInterceptionArgs)>. If several aspects are applied to the property (or to the property setter), <xref:PostSharp.Aspects.LocationInterceptionArgs.GetCurrentValue> will go through the next aspect in the chain of invocation. 

PostSharp also provides a mechanism to set the underlying property in a getter via the <xref:PostSharp.Aspects.LocationInterceptionArgs.SetNewValue(System.Object)> method of <xref:PostSharp.Aspects.LocationInterceptionArgs>. This could be used, for example, to ensure that a default value is assigned to the underlying property if there is currently no value. The following snippet shows a modified <xref:PostSharp.Aspects.LocationInterceptionAspect.OnGetValue(PostSharp.Aspects.LocationInterceptionArgs)> method that gets the current underlying value and sets a default value if the current value is null: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/UnderlyingValue.cs#on-get-value)]

With both methods in the aspect, a program that reads the `Name` property, sets it to `"John"` twice, and reads it again prints the following output:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/UnderlyingValue.txt)]


## Intercepting fields

One benefit of implementing a <xref:PostSharp.Aspects.LocationInterceptionAspect> is that it can be applied directly to fields, allowing for reads and writes to those fields to be intercepted, just like with properties. 

To apply a <xref:PostSharp.Aspects.LocationInterceptionAspect> implementation to a field, set it as an attribute on the field, as you do for a property: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptField.cs#customer)]

With the attribute applied to the name field, all attempts to get and set that field will be intercepted by `StringChecker` in its <xref:PostSharp.Aspects.LocationInterceptionAspect.OnGetValue(PostSharp.Aspects.LocationInterceptionArgs)> and <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> methods. 

Note that when a <xref:PostSharp.Aspects.LocationInterceptionAspect> is added to a field, the field is replaced by a property of the same name and visibility. The field itself is renamed and made private. 

The following output shows that the get and set operations on the `name` field are intercepted, and that `name` is now a public property:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/InterceptField.txt)]


## Getting the property or field being accessed

Information about the property or field being intercepted can be obtained through the <xref:PostSharp.Aspects.LocationInterceptionArgs> via its <xref:PostSharp.Aspects.LocationInterceptionArgs.Binding> property. 

Here you will find <xref:PostSharp.Aspects.ILocationBinding.LocationInfo>, which can represent a <xref:PostSharp.Reflection.LocationInfo.FieldInfo>, a <xref:PostSharp.Reflection.LocationInfo.PropertyInfo>, or a <xref:PostSharp.Reflection.LocationInfo.ParameterInfo> (although <xref:PostSharp.Aspects.LocationInterceptionAspect> cannot be added to parameters). 

One use for this is to reflect the property name whenever a property is changed. In the following example, we have an `Entity` class that implements <xref:System.ComponentModel.INotifyPropertyChanged> and a public `OnPropertyChanged` method that allows notifications to be made whenever a property is changed. 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationNotifyPropertyChanged.cs#entity)]

With the ability to invoke an `OnPropertyChanged` event, we can create a <xref:PostSharp.Aspects.LocationInterceptionAspect> that invokes this event when setting a value and passes in the property name from the underlying <xref:PostSharp.Reflection.LocationInfo.PropertyInfo> object: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationNotifyPropertyChanged.cs#aspect)]

> [!NOTE]
> This example is a simplistic implementation of the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect. For a production-ready implementation, see <xref:inotifypropertychanged>. 

The <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> attribute restricts the aspect to properties, so that it is not applied to fields, including the backing fields of automatic properties, when it is added to a class.

This aspect can then be applied to the `Customer` class, which derives from `Entity`: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationNotifyPropertyChanged.cs#customer)]

Now when the `Name` property is changed, the `NotifyPropertyChanged` aspect will invoke the `Entity.OnPropertyChanged` method, passing in the property name retrieved from its underlying property. When the program sets the `Name` property twice to the same value, only the first change raises the event:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationNotifyPropertyChanged.txt)]


## Reacting to initialization of instance fields and properties

C# and VB.NET allow fields and properties to be initialized on the same line as their declaration instead of in the constructor. This is called the *initialization* of the field or property. While the initialization of static fields and properties can be intercepted by <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)>, this method will not be called while initializing instance fields or properties. The reason for that limitation is that the <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> method requires a valid `this` object, which we do not have because inline initialization is performed before the base constructor has been called. 

The initialization of instance fields or properties cannot be intercepted. However, you can react to their initial value by implementing the <xref:PostSharp.Aspects.LocationInterceptionAspect.OnInstanceLocationInitialized(PostSharp.Aspects.LocationInitializationArgs)> advice. This advice will be invoked after the completion of the base constructor and before the execution of your own constructor. 


## Example

Suppose you have an item in an auction and you want to take an action each time the price of the item changes. You have the following class:

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationInitialization.cs#item)]

And you write your <xref:PostSharp.Aspects.LocationInterceptionAspect>: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationInitialization.cs#aspect)]

Then the following code snippet:

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationInitialization.cs#program)]

will have this output:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/LocationInitialization.txt)]

See the Remarks section of <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> and <xref:PostSharp.Aspects.LocationInterceptionAspect.OnInstanceLocationInitialized(PostSharp.Aspects.LocationInitializationArgs)> for more details. 

## See Also

**Reference**

<xref:PostSharp.Aspects.LocationInterceptionAspect>
<br><xref:PostSharp.Aspects.LocationInterceptionAspect.OnGetValue(PostSharp.Aspects.LocationInterceptionArgs)>
<br><xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)>
<br><xref:PostSharp.Aspects.LocationInterceptionAspect.OnInstanceLocationInitialized(PostSharp.Aspects.LocationInitializationArgs)>
<br><xref:PostSharp.Aspects.LocationInterceptionArgs>
<br><xref:PostSharp.Aspects.LocationInitializationArgs>
<br><xref:PostSharp.Reflection.LocationInfo>
<br><xref:PostSharp.Reflection.LocationInfo.FieldInfo>
<br><xref:PostSharp.Reflection.LocationInfo.PropertyInfo>
<br><xref:PostSharp.Reflection.LocationInfo.ParameterInfo>
<br><xref:System.ComponentModel.INotifyPropertyChanged>
<br><xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>

**Other Resources**

<xref:attribute-multicasting>
<br><xref:inotifypropertychanged>
