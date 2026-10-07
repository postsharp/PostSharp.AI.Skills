---
uid: attribute-multicasting
redirect_from:
  - custompatterns/aspects/applying/attribute-multicasting
title: "Adding Aspects to Multiple Declarations Using Attributes"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document describes how to use PostSharp's custom attribute multicasting to apply aspects to multiple declarations in an application code, including applying to all class members, all types in a namespace, excluding some members, and filtering by class visibility or method modifiers."
---
# Adding Aspects to Multiple Declarations Using Attributes

After you write an aspect, you must apply it to the application code so that it is used. There are several ways to do this. This article describes one of them: custom attribute multicasting. Other ways include XML multicasting (see the section <xref:xml-multicasting>) and dynamic aspect providers (see the section <xref:iaspectprovider>). 


## Applying to all members of a class

To apply a method-level aspect, you can place an attribute on each method. As your codebase grows, this approach becomes tedious. You need to remember to add the attribute to all methods of the class. If you have hundreds of classes, you may have thousands of methods to which you must add the aspect attribute manually. This approach does not scale. Fortunately, there is an easier way. Instead of applying your aspect to each method, you can add that attribute to the class, and PostSharp ensures that the aspect is applied to all methods of that class.

```csharp
[OurLoggingAspect] 
public class CustomerServices
```

You can also add a location-level aspect to a class which applies it to all properties and all fields in that class. Note that this includes backing fields of auto-implemented properties.


## Applying an aspect to all types in a namespace

Even though you do not have to apply an aspect to every method in every class of your application, adding the aspect attribute to every class could still be a large task. To apply your aspect broadly, you can use the <xref:PostSharp.Extensibility.MulticastAttribute> of PostSharp. 

The <xref:PostSharp.Extensibility.MulticastAttribute> is a special attribute that applies other attributes throughout your codebase. To use it, follow these steps:

1. Open the `AssemblyInfo.cs`, or create a new file `GlobalAspects.cs` if you prefer to keep things separate (the name of this file does not matter). 


2. Add an `[assembly:]` attribute that references the aspect you want to apply. 


3. Add the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetTypes> property to the aspect attribute and define the namespace that you would like the aspect applied to. 

    ```csharp
    [assembly: OurLoggingAspect(AttributeTargetTypes="OurCompany.OurApplication.Controllers.*")]
    ```


This one line of code is the equivalent of adding the aspect attribute to every class in the desired namespace.

> [!NOTE]
> When setting the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetTypes> you can use wildcards (`*`) to indicate that all sub-namespaces should have the aspect applied to them. It is also possible to indicate the targets of the aspect using `regex`. Add `"regex:"` as a prefix to the pattern you wish to use for matching. 


## Excluding an aspect from some members

Multicasting an attribute can apply the aspect to many targets. To restrict where the aspect is attached, use <xref:PostSharp.Extensibility.MulticastAttribute.AttributeExclude>. 

```csharp
[assembly: OurLoggingAspect(AttributeTargetTypes="OurCompany.OurApplication.Controllers.*", AttributePriority = 1)] 
[assembly: OurLoggingAspect(AttributeTargetMembers="Dispose", AttributeExclude = true, AttributePriority = 2)]
```

In the example above, the first multicast line indicates that the `OurLoggingAspect` should be attached to all methods in the `Controllers` namespace. The second multicast line indicates that the `OurLoggingAspect` should not be applied to any method named `Dispose`. 

> [!NOTE]
> Notice the <xref:PostSharp.Extensibility.MulticastAttribute.AttributePriority> property that is set in both of the multicast lines. Since there is no guarantee that the compiler will apply the attributes in the order you have specified in the code, it is necessary to declare an order to ensure processing is completed as desired. 
> In this case, the `OurLoggingAspect` is first applied to all methods in the `Controllers` namespace. After that is completed, the second multicast of `OurLoggingAspect` is performed, which then excludes the aspect from methods named `Dispose`. 

See <xref:multicast-override> for more details about excluding and overriding aspects. 


## Filtering by class visibility

After you apply an aspect to all classes in a namespace and its sub-namespaces, you may need to restrict it. For example, you may want to apply your aspect only to public classes. To do this:

1. Add the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetTypeAttributes> property to the aspect attribute. 


2. Set the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetTypeAttributes> value to `Public`. 

    ```csharp
    [assembly: OurLoggingAspect(AttributeTargetTypes="OurCompany.OurApplication.Controllers.*",  
                                AttributeTargetTypeAttributes = MulticastAttributes.Public)]
    ```


By combining <xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetTypeAttributes> values, you can create the combinations that you need. 

> [!NOTE]
> <xref:PostSharp.Extensibility.MulticastAttributes> flags fall into categories: visibility, scope, abstraction, virtuality, implementation, literality, generation, and parameter. You only need to specify the categories on which you want to put a restriction. For the categories you leave empty, PostSharp uses the flags of the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> of the aspect.


## Filtering by method modifiers

Filtering at the class level may not be granular enough for your needs. Aspects can be attached at the method level, and you may want to control filtering on these aspects as well. The following steps apply aspects only to methods marked as virtual:

1. Add the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetMemberAttributes> property to the aspect attribute. 


2. Set the <xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetMemberAttributes> value to `Virtual`. 

    ```csharp
    [assembly: OurLoggingAspect(AttributeTargetTypes="OurCompany.OurApplication.Controllers.*", AttributeTargetMemberAttributes = MulticastAttributes.Virtual)]
    ```


Using this technique you can apply a method-level aspect, or stop it from being applied, based on whether keywords such as `static`, `abstract` and `virtual` are present.


## Programmatic filtering

There are situations where you will want to filter in a way that is not based on class or method declarations. You may want to apply an aspect only if a class inherits from a specific class or implements a certain interface. PostSharp offers two approaches for this.

The easiest way is to override the <xref:PostSharp.Aspects.Aspect.CompileTimeValidate(System.Object)> method of your aspect class, where you can perform your custom filtering. This is the opt-out approach. If the <xref:PostSharp.Aspects.Aspect.CompileTimeValidate(System.Object)> method returns `false` without throwing an exception, the target candidate is ignored. See the section <xref:aspect-validation> for details. 

The second approach is opt-in. See the section <xref:iaspectprovider> for details. 

## See Also

**Reference**

<xref:PostSharp.Extensibility.MulticastAttribute>
<br><xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetTypes>
<br><xref:PostSharp.Extensibility.MulticastAttribute.AttributeExclude>
<br><xref:PostSharp.Extensibility.MulticastAttribute.AttributePriority>
<br><xref:PostSharp.Extensibility.MulticastAttribute.AttributeTargetTypeAttributes>
<br><xref:PostSharp.Aspects.Aspect.CompileTimeValidate(System.Object)>
<br><xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.PersistMetaData>

**Other Resources**

<xref:aspect-validation>
<br><xref:iaspectprovider>
<br><xref:xml-multicasting>
