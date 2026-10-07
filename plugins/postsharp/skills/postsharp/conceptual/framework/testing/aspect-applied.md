---
uid: testing-application
redirect_from:
  - custompatterns/aspects/testing/testing-application
title: "Testing that an Aspect has been Applied"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides guidelines on how to test if an aspect has been applied to the expected targets in PostSharp, including testing the aspect behavior and checking for the aspect's custom attribute."
---
# Testing that an Aspect has been Applied

The previous section showed how to test the aspect behavior itself. This article shows how to test that the aspect has been applied to the expected set of targets. This is also called *testing the pointcut*. 


## Why test that the aspect has been properly applied?

You may need to test whether an aspect has been applied to specific targets for one of the following reasons:

* The aspect is applied using non-trivial regular expressions with <xref:PostSharp.Extensibility.MulticastAttribute>. 

* The aspect is silently filtered out using <xref:PostSharp.Aspects.MethodLevelAspect.CompileTimeValidate(System.Reflection.MethodBase)>. 

* The aspect is applied using an <xref:PostSharp.Aspects.IAspectProvider>. 


## Testing that the aspect behavior is exhibited

The most obvious way to test that the aspect has been applied to an element of code is to execute that code and ensure that the code actually exhibits the aspect behavior. This approach does not differ from the one described in section <xref:simple-tests>. 


## Testing that the aspect custom attribute is present

You can check that an aspect has been applied to a target by reflecting the custom attributes present on this element of code.

However, custom attributes representing aspects are stripped by default. If you want PostSharp to emit custom attributes, follow the instructions in <xref:multicast-reflection>. 

> [!NOTE]
> Aspects added by <xref:PostSharp.Aspects.IAspectProvider> are not represented by custom attributes, so their presence cannot be tested by this approach. 


## Parsing the PostSharp symbol file

PostSharp generates a symbol file named *MyAssembly.pssym* in the output directory (for instance *bin\Debug*), where *MyAssembly* is the name of the assembly. In theory, you could use this file to determine which elements of code have been modified by aspects in your project. 

> [!CAUTION]
> The PostSharp symbol file format is undocumented and unsupported. Therefore, the PostSharp support team cannot answer questions related to this file format.


