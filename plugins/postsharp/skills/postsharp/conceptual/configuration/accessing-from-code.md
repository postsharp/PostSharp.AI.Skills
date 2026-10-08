---
uid: configuration-consuming
redirect_from:
  - deploymentconfiguration/configuration/configuration-consuming
title: "Accessing Configuration from Source Code"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to access configuration elements from user code in PostSharp, including properties and custom sections, using specific methods."
---
# Accessing Configuration from Source Code

Although PostSharp or its add-ins consume most configuration settings, it is sometimes useful to access configuration elements from user code. The *PostSharp.dll* library gives access to both configuration properties and extension configuration elements. 


## Accessing properties

You can read the value of any PostSharp property by including it in an XPath expression and evaluating it using the <xref:PostSharp.Extensibility.IProject.EvaluateExpression(System.String)> method of <xref:PostSharp.Extensibility.PostSharpEnvironment.CurrentProject>: 

```csharp
string value = PostSharpEnvironment.CurrentProject.EvaluateExpression("{$PropertyName}");
```

For details regarding expressions, see <xref:configuration-xpath>. 


## Accessing custom sections

You can get a list of custom sections of a given name and namespace by calling the <xref:PostSharp.Extensibility.IProject.GetExtensionElements(System.String,System.String)> method of <xref:PostSharp.Extensibility.PostSharpEnvironment.CurrentProject>: 

```csharp
IEnumerable<ProjectExtensionElement> elements = 
     PostSharpEnvironment.CurrentProject.GetExtensionElements( "MyElement", "uri:MyNamespace" );
```

Extension elements must be declared using the `SectionType` element. 

## See Also

**Other Resources**

<xref:configuration-xpath>
<br><xref:configuration-system>
