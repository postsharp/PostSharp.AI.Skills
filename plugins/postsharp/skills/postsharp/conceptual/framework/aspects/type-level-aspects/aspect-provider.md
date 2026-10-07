---
uid: aspect-provider
redirect_from:
  - custompatterns/aspects/compositeaspects/aspect-provider
  - framework/aspects/composite-aspects/aspect-provider
title: "Adding Aspects Dynamically"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to add aspects dynamically in PostSharp using IAspectProvider, especially when it is impossible to add aspects declaratively to the source code."
---
# Adding Aspects Dynamically

In addition to providing advices, an aspect can provide other aspects dynamically using <xref:PostSharp.Aspects.IAspectProvider>. This allows aspect developers to address situations where it is not possible to add aspects declaratively (using custom attributes) to the source code. For instance, aspects can be provided on the basis of a complex analysis of the target assembly using <xref:System.Reflection>, or by reading an XML file. 

For details about <xref:PostSharp.Aspects.IAspectProvider>, see <xref:iaspectprovider>. 

## See Also

**Reference**

<xref:PostSharp.Aspects.IAspectProvider>

**Other Resources**

<xref:iaspectprovider>
<br><xref:advice-provider>
<br><xref:pointcuts>
