---
uid: iaspectprovider
redirect_from:
  - custompatterns/aspects/applying/iaspectprovider
title: "Adding Aspects Programmatically Using IAspectProvider"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to add aspects programmatically using the IAspectProvider interface in PostSharp, including creating aspect graphs and initializing aspects at run time."
---
# Adding Aspects Programmatically Using IAspectProvider
In some situations, you may want to implement an aspect as part of a larger pattern. For example, you may want to add an aspect, implement an interface and dynamically inject some logic into the target code. In those situations, apply an aspect to the target code and let that aspect add other aspects to other elements of code.

The following steps show the implementation.

1. Create an aspect that implements the <xref:PostSharp.Aspects.IAspectProvider> interface. 

    ```csharp
    [PSerializable]
    public class ProviderAspect : TypeLevelAspect, IAspectProvider 
     { 
        public IEnumerable<AspectInstance> ProvideAspects(object targetElement) 
        { 
            throw new System.NotImplementedException(); 
        } 
     }
    ```


2. Cast the target object parameter to the type that will be targeted by this aspect: `Assembly`, `Type`, `FieldInfo`, `MethodBase`, `PropertyInfo`, `EventInfo`, `ParameterInfo` or `LocationInfo`. 

    ```csharp
    public IEnumerable<AspectInstance> ProvideAspects(object targetElement) 
     { 
        Type type = (Type) targetElement; 
     
        throw new NotImplementedException(); 
     }
    ```


3. Return an <xref:PostSharp.Aspects.AspectInstance> of the aspect type you want for every target element of code. The target elements must be inside the element of code to which the provider aspect is applied. For instance, a type-level provider can add aspects to the methods of its own type, but not to the methods of other types. 

    ```csharp
    public IEnumerable<AspectInstance> ProvideAspects(object targetElement) 
     { 
         Type type = (Type)targetElement; 
     
         return type.GetMethods( BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly ).Select( 
            m => new AspectInstance(m, new LoggingAspect() ) ); 
     
     }
    ```


This aspect now adds aspects dynamically at build time. The <xref:PostSharp.Aspects.IAspectProvider> interface is usually reserved for situations where you implement a larger design pattern. For example, you can use it to implement an aspect that creates the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> pattern across a large number of locations in your codebase. It is more than many situations require. Use it only for aspects that implement complex patterns. 

> [!NOTE]
> To read more about <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>, see <xref:inotifypropertychanged-customization>. 

> [!NOTE]
> Aspects provided by <xref:PostSharp.Aspects.IAspectProvider> are initialized like other aspects: PostSharp calls their `CompileTimeInitialize` method at build time and their `RunTimeInitialize` method at run time. 


## Creating Graphs of Aspects

It is common that aspects provided by <xref:PostSharp.Aspects.IAspectProvider> (child aspects) form a complex object graph. For instance, child aspects may contain a reference to the parent aspect. 

PostSharp serializes object graphs that are instantiated at build time, so they can be used at run time. In other words, if you store a reference to another aspect in a child aspect, you will be able to use this reference at run time.

## Extension Block Members

C# 14 introduced extension members. Both <xref:PostSharp.Aspects.IAspectProvider> and <xref:PostSharp.Aspects.Advices.IAdviceProvider> can be used on extension member implementation methods (members of the class containing extension blocks).

Targeting extension block metadata types (nested types emitted by the C# compiler) and all their members will result in an error `LA0167`. You can use <xref:PostSharp.Reflection.ReflectionHelper.IsExtensionBlockMetadata> to identify extension block metadata members if filtering is needed.

## See Also

**Reference**

<xref:PostSharp.Aspects.IAspectProvider>
<br><xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>
<br><xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)>
<br><xref:PostSharp.Aspects.AspectInstance>

**Other Resources**

<xref:inotifypropertychanged>
<br><xref:inotifypropertychanged-customization>
<br>[C# 14 Extension Members - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/extension)
<br>[Exploring extension members - .NET Blog](https://devblogs.microsoft.com/dotnet/csharp-exploring-extension-members/)
