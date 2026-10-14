---
uid: disposable
redirect_from:
  - model/aggregatable/disposable
title: "Automatically Disposing Children Objects (Disposable)"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to automatically dispose of objects in hierarchies using the PostSharp.Patterns.Model.DisposableAttribute aspect in PostSharp. It covers implementing IDisposable interface, customizing the Dispose logic, and provides coding examples."
---
# Automatically Disposing Children Objects (Disposable)

When you are working with hierarchies of objects, you sometimes run into situations where you need to properly dispose of an object. Not only will you need to dispose of that object, but you likely will need to walk the object tree and recursively dispose of children of that object. To do this, we typically implement the <xref:System.IDisposable> pattern and manually code the steps required to shut down the desired objects, and call the <xref:System.IDisposable.Dispose> method on other children objects. This cascading of disposals takes a lot of effort and it is prone to mistakes and omissions. 

The <xref:PostSharp.Patterns.Model.DisposableAttribute> aspect relies on the <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect and, as a result, is able to make use of the <xref:PostSharp.Patterns.Model.IAggregatable.VisitChildren(PostSharp.Patterns.Model.ChildVisitor,PostSharp.Patterns.Model.ChildVisitorOptions,System.Object)> method to cascade disposals through child objects. 


## Disposing of object trees


### Implementing the IDisposable interface automatically

1. Add a reference to the `PostSharp.Patterns.Model` package. 


2. On the top-level object, add the <xref:PostSharp.Patterns.Model.DisposableAttribute>. 


3. Annotate the object model as described in <xref:aggregatable-adding>. 


> [!NOTE]
> Fields that are marked as children but are assigned to an object that does not implement <xref:System.IDisposable> (either manually or through <xref:PostSharp.Patterns.Model.DisposableAttribute>) will simply be ignored during disposal. 

> [!NOTE]
> Items of child collections will be automatically disposed of as well unless items of child collections are considered as references. See <xref:advisable-collections> for details. 


### Example

In this example, the `HomeMadeLogger` class has two fields, `_stream` and `_textWriter`, which should also be disposed of when the `HomeMadeLogger` is disposed of. 

[!code-csharp[](../../code/samples/Model/Aggregatable/DisposableLogger.cs#example)]

The `_stream` and `_textWriter` child objects will now have their `Dispose()` method called automatically when the `HomeMadeLogger` is disposed of. Since both the `_stream` and `_textWriter` objects are framework types that already implement <xref:System.IDisposable>, adding the <xref:PostSharp.Patterns.Model.DisposableAttribute> aspect to those object types is not necessary. 

PostSharp disposes of the children in the reverse order of their declaration. In this example, `_textWriter` is disposed of first, so it can flush its buffer to `_stream`, which is still open. If `_stream` were declared after `_textWriter`, the writer would throw an <xref:System.ObjectDisposedException> when disposed of. 

The following program writes a message and disposes of the logger. The message is in the file:

[!code-text[](../../code/samples/Model/Aggregatable/DisposableLogger.txt)]


## Customizing the Dispose logic

There will be times when you have objects that need custom disposal logic. At the same time, you may want to implement a parent-child relationship and make use of the <xref:PostSharp.Patterns.Model.DisposableAttribute>. PostSharp allows you to combine custom and automatic logic. 

To add your own logic to the `Dispose` method, create a method with exactly the following signature: 

[!code-csharp[](../../code/samples/Model/Aggregatable/DisposableCustom.cs#dispose)]

> [!WARNING]
> The <xref:PostSharp.Patterns.Model.DisposableAttribute> aspect does not automatically dispose of the object when it is garbage collected. That is, the aspect does not implement a destructor. If you need a destructor, you have to do it manually and invoke the `Dispose`. 


### Example

In the following example, we are customizing the Dispose pattern to expose the `IsDisposed` property: 

[!code-csharp[](../../code/samples/Model/Aggregatable/DisposableCustom.cs#example)]

Once you have done this, PostSharp will properly run your custom Dispose logic as well as running any of the parent and child implementations of the <xref:PostSharp.Patterns.Model.DisposableAttribute> that exist for the object. 

[!code-text[](../../code/samples/Model/Aggregatable/DisposableCustom.txt)]


