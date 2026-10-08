---
uid: multicast-reflection
redirect_from:
  - custompatterns/aspects/applying/multicast-reflection
title: "Reflecting Aspect Instances at Run Time"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to make attribute multicasting in PostSharp visible at runtime using the PersistMetaData property in the MulticastAttributeUsageAttribute class."
---
# Reflecting Aspect Instances at Run Time
Attribute multicasting was designed primarily as a mechanism to add aspects to a program. Most of the time, the custom attribute representing an aspect can be removed after the aspect has been applied.

By default, if you add an aspect to a program and look at the resulting program using a disassembler or <xref:System.Reflection>, you will not find these corresponding custom attributes. 

If you need your aspect (or any other multicast attribute) to be reflected by <xref:System.Reflection> or any other tool, you have to set the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.PersistMetaData> property to `true`. 

For instance:

```csharp
[MulticastAttributeUsage( MulticastTargets.Class, PersistMetaData = true )]
        public class TagAttribute : MulticastAttribute
        {
          public string Tag;
        }
```

> [!NOTE]
> Multicasting of attributes is not limited to PostSharp aspects. You can multicast any custom attribute in your codebase in the same way as shown here. If a custom attribute is multicast with the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.PersistMetaData> property set to `true`, the compiled code, when reflected, looks as if you had manually added the custom attribute in all of the locations. 


