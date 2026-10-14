---
uid: inotifypropertychanged-add
redirect_from:
  - model/notifypropertychanged/inotifypropertychanged-add
title: "Implementing INotifyPropertyChanged"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to implement the INotifyPropertyChanged interface using PostSharp, including adding the NotifyPropertyChanged aspect, and accessing the INotifyPropertyChanged interface from your code."
---
# Implementing INotifyPropertyChanged

This section shows how to make your class implement the <xref:System.ComponentModel.INotifyPropertyChanged> interface automatically by using the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect. 


## Adding the NotifyPropertyChanged aspect


To add the `NotifyPropertyChanged` aspect to a class:

1. Use NuGet Package Manager to add the `PostSharp.Patterns.Model` package to your project. 


2. Import the <xref:PostSharp.Patterns.Model> namespace into your file. 


3. Add the `[NotifyPropertyChanged]` custom attribute to the class. 


All properties of the class now fire the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event whenever they are changed. You can prevent a property from automatically firing the `PropertyChanged` event with <xref:PostSharp.Patterns.Model.IgnoreAutoChangeNotificationAttribute>. 

Note that the `[NotifyPropertyChanged]` aspect is inherited. Changes of properties in all derived classes will automatically be notified. 

By using PostSharp to add <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> to your model classes you are able to eliminate most of the repetitive boilerplate coding tasks and code from the codebase. 

> [!NOTE]
> This procedure has added <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> to one class hierarchy. If you need to add the aspect to several classes in your codebase, consider using aspect multicasting. See <xref:attribute-multicasting> for details. We recommend being careful with multicasting and to add this aspect only to classes that really require the feature because this aspect has some performance and memory consumption overhead. 


### Example

[!code-csharp[](../../code/samples/Model/NotifyPropertyChanged/NpcAdd.cs#example)]


## Consuming the INotifyPropertyChanged interface

Since the <xref:System.ComponentModel.INotifyPropertyChanged> interface is implemented by PostSharp at build time after the compiler has completed, the interface will not be visible to IntelliSense, to other tools like ReSharper or to the compiler. The same is true for the `PropertyChanged` event. 

In many cases, this limitation does not matter because the interface is consumed from a framework (like WPF) that is not coupled with your project. However, in some situations, you may need to access the <xref:System.ComponentModel.INotifyPropertyChanged> interface. 

There are two ways to access the <xref:System.ComponentModel.INotifyPropertyChanged> interface from your code: 

* You can cast your object to <xref:System.ComponentModel.INotifyPropertyChanged>, for instance: 
    [!code-csharp[](../../code/samples/Model/NotifyPropertyChanged/NpcAdd.cs#cast)]

    If your tooling complains that the object does not implement the interface, you can first cast to `object`: 
    [!code-csharp[](../../code/samples/Model/NotifyPropertyChanged/NpcAdd.cs#object-cast)]


* You can use the <xref:PostSharp.Post.Cast``2(``0)> method. The benefit of using this method is that the cast operation is validated by PostSharp, so the build will fail if you try to cast an object that does not implement the <xref:System.ComponentModel.INotifyPropertyChanged> interface. For instance: 
    [!code-csharp[](../../code/samples/Model/NotifyPropertyChanged/NpcAdd.cs#post-cast)]


## See Also

**Reference**

<xref:System.ComponentModel.INotifyPropertyChanged>
<br><xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>

**Other Resources**

<xref:attribute-multicasting>
