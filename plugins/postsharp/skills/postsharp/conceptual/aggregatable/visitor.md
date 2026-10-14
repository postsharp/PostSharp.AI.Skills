---
uid: aggregatable-visitor
redirect_from:
  - model/aggregatable/aggregatable-visitor
title: "Enumerating Child Objects (Visitor)"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use the Aggregatable pattern in PostSharp to enumerate child objects of a parent, including a step-by-step guide and code examples."
---
# Enumerating Child Objects (Visitor)

After you have <xref:aggregatable-adding>, you will want to make use of it. 

Both the <xref:PostSharp.Patterns.Model.ChildAttribute> and <xref:PostSharp.Patterns.Model.ParentAttribute> can be used to declare parent-child relationships for other patterns such as Undo/Redo (<xref:PostSharp.Patterns.Recording.RecordableAttribute>) or threading models (<xref:PostSharp.Patterns.Threading.ImmutableAttribute>, <xref:PostSharp.Patterns.Threading.FreezableAttribute>, ...). 

You can also use the Aggregatable pattern from your own code. The functionalities of this pattern are exposed by the <xref:PostSharp.Patterns.Model.IAggregatable> interface, which all aggregatable objects automatically implement. This interface allows you to execute a Visitor method against all child objects of a parent. 

In the following example, we see how to implement recursive validation for an object model. We will assume that the `InvoiceLine` and `Address` classes implement an `IValidatable` interface. 


## Enumerating all child objects of a parent

1. Cast the parent object to the <xref:PostSharp.Patterns.Model.IAggregatable> interface. 

    [!code-csharp[](../../code/samples/Model/Aggregatable/AggregatableVisitor.cs#cast)]

    > [!NOTE]
    > The <xref:PostSharp.Patterns.Model.IAggregatable> interface will be injected into the `Invoice` class *after* compilation. Tools that are not aware of PostSharp may incorrectly report that the `Invoice` class does not implement the <xref:PostSharp.Patterns.Model.IAggregatable> interface. Instead of using the cast operator, you can also use the <xref:PostSharp.Post.Cast``2(``0)> method. This method is faster and safer than the cast operator because it is verified and compiled by PostSharp at build time. 

    > [!NOTE]
    > If you are attempting to access <xref:PostSharp.Patterns.Model.IAggregatable> members on either <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> or <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2> you will not be able to use the cast operator or the <xref:PostSharp.Post.Cast``2(``0)> method. Instead, you will have to use the <xref:PostSharp.Patterns.DynamicAdvising.DynamicAdvisingServices.QueryInterface``1(System.Object,System.Boolean)> method. 


2. Invoke the <xref:PostSharp.Patterns.Model.IAggregatable.VisitChildren(PostSharp.Patterns.Model.ChildVisitor,PostSharp.Patterns.Model.ChildVisitorOptions,System.Object)> method and pass a delegate to the method to be executed. 

    [!code-csharp[](../../code/samples/Model/Aggregatable/AggregatableVisitor.cs#visit)]

    > [!NOTE]
    > The visitor must return `true` to continue the enumeration and `false` to stop the enumeration. The <xref:PostSharp.Patterns.Model.IAggregatable.VisitChildren(PostSharp.Patterns.Model.ChildVisitor,PostSharp.Patterns.Model.ChildVisitorOptions,System.Object)> method returns `false` if the visitor stopped the enumeration. 

With an invoice that has a line whose amount is zero and a delivery address without a city, the program finds two errors:

[!code-text[](../../code/samples/Model/Aggregatable/AggregatableVisitor.txt)]


## See Also

**Reference**

<xref:PostSharp.Patterns.Model.AggregatableAttribute>
<br><xref:PostSharp.Patterns.Model.ParentAttribute>
<br><xref:PostSharp.Patterns.Model.ChildAttribute>
<br><xref:PostSharp.Patterns.Model.IAggregatable>
<br><xref:PostSharp.Patterns.Model.IAggregatable.VisitChildren(PostSharp.Patterns.Model.ChildVisitor,PostSharp.Patterns.Model.ChildVisitorOptions,System.Object)>
<br><xref:PostSharp.Post.Cast``2(``0)>
<br><xref:PostSharp.Patterns.DynamicAdvising.DynamicAdvisingServices.QueryInterface``1(System.Object,System.Boolean)>
