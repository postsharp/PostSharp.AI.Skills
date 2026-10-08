---
uid: module-initializer
title: "Executing Code Just After the Assembly is Loaded"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use the PostSharp.Aspects.ModuleInitializerAttribute attribute to implement module initializers in C# similar to Visual Basic."
---
# Executing Code Just After the Assembly is Loaded

The CLR supports *module initializers*: code that runs when the assembly is loaded, before any other code of the assembly. C# did not expose this feature before C# 9. The <xref:PostSharp.Aspects.ModuleInitializerAttribute> attribute allows you to have module initializers in any C# version. 


## Adding a module initializer to your project

1. Create a public or internal static method that has no parameters and no return value. The method cannot be generic, and the type declaring the method cannot have generic parameters.


2. Add the <xref:PostSharp.Aspects.ModuleInitializerAttribute> attribute to this method. 


You can add several module initializers to a project. Module initializers are executed in the order you specify in the attribute constructor, smallest value first.

## See Also

**Reference**

<xref:PostSharp.Aspects.ModuleInitializerAttribute>
