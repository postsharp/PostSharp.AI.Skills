---
uid: resource-introduction
redirect_from:
  - custompatterns/aspects/tutorials/resource-introduction
title: "Introducing Managed Resources"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document introduces PostSharp's AssemblyLevelAspect, which allows for programmatically adding resources at compile time in .NET, adding flexibility in determining which resources get embedded and how. It provides a coding example demonstrating the process."
---
# Introducing Managed Resources

Embedding resources in .NET allows for data to be packaged together with your code in an assembly. Resources are normally specified at design time and then embedded by the compiler during build time.

PostSharp's <xref:PostSharp.Aspects.AssemblyLevelAspect> adds additional flexibility by allowing you to programmatically add resources at compile time. In doing so, you can add logic, and therefore flexibility, in determining which resources are embedded and how. For example, you could use this feature to encrypt a resource just before embedding it into your assembly. 


## Introducing resources

In the following example, we create an assembly-level aspect that builds a string during compilation and stores it in the current assembly as a resource. The example then shows that this string can be retrieved from the assembly at run time.


1. Create a class called `AddBuildInfoAspect` that derives from <xref:PostSharp.Aspects.AssemblyLevelAspect>. Also implement <xref:PostSharp.Aspects.IAspectProvider>, which exposes the <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> method. The <xref:PostSharp.Aspects.IAspectProvider.ProvideAspects(System.Object)> method will be called once by PostSharp, providing access to assembly information and allowing for a resource to be programmatically added to the assembly: 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/ResourceIntroduction.cs#aspect)]

    In this example, the `targetElement` object passed in is cast to an `Assembly` object from which the assembly name is retrieved. The code then builds a string that contains the assembly name, and converts this string to a byte array. A real aspect could store other information that is only known at build time, for instance the build date or the name of the build machine. The byte array is then stored along with a name for the data in PostSharp’s <xref:PostSharp.Aspects.ManagedResourceIntroductionAspect> object, and returned via an <xref:PostSharp.Aspects.AspectInstance>. PostSharp then embeds the resource into the current assembly. 


2. Add the `AddBuildInfoAspect` attribute to the assembly, for instance in the AssemblyInfo.cs file: 

    [!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/ResourceIntroduction.cs#assembly)]


With this code in place, the assembly embeds the string as a resource into itself during compilation.

The following code demonstrates how to retrieve the data at run time:

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/ResourceIntroduction.cs#program)]

This will display the following line in the console window:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/Tutorials/ResourceIntroduction.txt)]

## See Also

**Reference**

<xref:PostSharp.Aspects.AssemblyLevelAspect>
<br><xref:PostSharp.Aspects.IAspectProvider>
<br><xref:PostSharp.Aspects.ManagedResourceIntroductionAspect>
<br><xref:PostSharp.Aspects.AspectInstance>

**Other Resources**

<xref:iaspectprovider>
