---
uid: aggregatable
redirect_from:
  - model/aggregatable
title: "Parent/Child, Visitor and Disposable"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses the concept of parent-child relationship in object-oriented design, its implementation in PostSharp, and the three types of object relationships in UML specification: Aggregation, Composition, and Association."
---
# Parent/Child, Visitor and Disposable

The parent-child relationship is a foundational concept of object-oriented design. There are three kinds of object relationships in the UML specification:

* *Aggregation* is the parent-child (also named whole-part) relationship. It is implemented in PostSharp by the <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect described in <xref:aggregatable-adding>. 

* *Composition* is an aggregation relationship where the parent controls the lifetime of its children. It is implemented in PostSharp by the <xref:PostSharp.Patterns.Model.DisposableAttribute> aspect pattern, which relies on the <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect. For details, see <xref:disposable>. 

* *Association* is a simple reference between two objects. 

Despite its importance, C# and VB have no keyword to represent aggregation. All C# and VB object references correspond to an association. Therefore, most applications and frameworks tend to re-implement the aggregation relationship, resulting in boilerplate code and defects. For instance, UI frameworks such as WinForms and WPF rely on a parent-child structure.

PostSharp implements the Aggregatable pattern by using the <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect, together with the <xref:PostSharp.Patterns.Model.ChildAttribute>, <xref:PostSharp.Patterns.Model.ReferenceAttribute> and <xref:PostSharp.Patterns.Model.ParentAttribute> custom attributes. 

The Aggregatable pattern is used by other PostSharp aspects, including all threading models (<xref:PostSharp.Patterns.Threading.ThreadAwareAttribute>), <xref:PostSharp.Patterns.Model.DisposableAttribute> and <xref:PostSharp.Patterns.Recording.RecordableAttribute>. You can also use the aspect to automatically implement a parent-child relationship in your own code. 


## In this chapter

| Section | Description |
|---------|-------------|
| <xref:aggregatable-adding> | This section shows how to prepare a class so that it can participate in a parent-child relationship. |
| <xref:aggregatable-visitor> | This section describes how to enumerate the children of an object thanks to the visitor pattern. |
| <xref:disposable> | This section shows how to automatically implement the <xref:System.IDisposable> interface so that children objects are disposed when the parent object is disposed.  |
| <xref:advisable-collections> | This section covers advanced topics related to collections in aggregatable object models. |


