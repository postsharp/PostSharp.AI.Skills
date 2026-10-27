---
uid: programmatic-tooltip
redirect_from:
  - custompatterns/aspects/intellisense/programmatic-tooltip
title: "Pushing Information to Visual Studio Tools for Metalama and PostSharp Programmatically"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses how to use the IWeavingSymbolsService in Visual Studio Tools for Metalama and PostSharp to push information from your aspect at build time."
---
# Pushing Information to Visual Studio Tools for Metalama and PostSharp Programmatically

The <xref:PostSharp.Extensibility.IWeavingSymbolsService> service allows you to push information from your aspect, at build time, to Visual Studio Tools for Metalama and PostSharp. 

This service can be used in the following scenarios:

* Adding text to the CodeLens and IntelliSense tooltip of a declaration.

* Adding code saving information.

* Adding an annotation that tells Visual Studio Tools for Metalama and PostSharp to consider that a declaration has been decorated with a custom attribute. This annotation is then taken into account by the analytic engine that powers the real-time quick actions and diagnostics of Visual Studio Tools for Metalama and PostSharp. For instance, the <xref:PostSharp.Patterns.Model.TypeAdapters.FieldRule> facility uses this feature. 

To get an instance of this service, call the <xref:PostSharp.Extensibility.IProject.GetService``1(System.Boolean)> method of `PostSharpEnvironment.CurrentProject`. 

The annotations that an aspect pushes are written to the weaving symbol file. `postsharp introspect list annotations` and `postsharp introspect show declaration` show them outside Visual Studio, which is a way to check them after a build. See <xref:introspect>.

## See Also

**Other Resources**

<xref:introspect>
<br>
