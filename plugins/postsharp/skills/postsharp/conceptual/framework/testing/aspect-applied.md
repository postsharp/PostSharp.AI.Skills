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


## Inspecting aspects from the CLI

The `postsharp introspect` commands of the `postsharp` command-line tool tell which aspects apply to which declarations, including the aspects added by <xref:PostSharp.Aspects.IAspectProvider>, which no custom attribute represents. They read the weaving symbol file that PostSharp writes next to the assembly, *MyAssembly.pssym*, and never build the project. Use them rather than parsing the file.

A test of the pointcut can therefore run after the build, outside of the tested assembly. For instance, it can check that the expected number of methods is logged:

```powershell
postsharp introspect list aspect-instances --where "aspectClass/shortName eq 'Log'" --count
```

The output is JSON and the exit code is 0 when the query runs, so a script can compare the result with an expected value. To install the tool and learn the other commands, see <xref:introspect>.

> [!CAUTION]
> The format of the PostSharp symbol file is undocumented and can change. The commands, not the file, are supported.


