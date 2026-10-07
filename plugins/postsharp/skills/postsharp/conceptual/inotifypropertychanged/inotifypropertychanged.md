---
uid: inotifypropertychanged
redirect_from:
  - model/notifypropertychanged/inotifypropertychanged
title: "INotifyPropertyChanged"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses the use of PostSharp's INotifyPropertyChanged attribute, which simplifies and optimizes the process of binding objects to the UI by automatically implementing property change notifications."
---
# INotifyPropertyChanged

Binding objects to the UI is a large and tedious task. You must implement <xref:System.ComponentModel.INotifyPropertyChanged> on every property that needs to be bound. You need to ensure that the underlying property setter correctly raises events so that the View knows that changes have occurred. The larger your codebase, the more work there is. You can partially eliminate all of this repetitive code by pushing some of the functionality to a base class that each Model class inherits from. It still does not eliminate all of the repetition. 

PostSharp can completely eliminate all of that repetition for you. PostSharp's <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> has the following features and benefits: 

* Supports computed properties.

* Supports computed properties referencing child objects, for example `string FullName => this.model.FirstName + " " + this.model.LastName`. 

* Raises the `PropertyChanged` event at the right moment, when all class invariants are valid. 

* Fully customizable. Can be integrated with other MVVM frameworks.

* High-performance. Almost as fast as handwritten code.


## In this chapter

| Topic | Description |
|-------|-------------|
| <xref:inotifypropertychanged-add> | This section shows how to automatically implement the <xref:System.ComponentModel.INotifyPropertyChanged> interface in a class by using the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect.  |
| <xref:inotifypropertychanged-dependencies> | This section describes how to handle dependencies that cross several objects, as when a view-model object is dependent on properties of a model object. |
| <xref:inotifypropertychanging> | This section documents how to implement the <xref:System.ComponentModel.INotifyPropertyChanging> interface for components which need to be signaled *before* a property value is changed.  |
| <xref:inotifypropertychanged-frameworks> | This section shows examples of using the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect with some of the popular UI frameworks.  |
| <xref:inotifypropertychanged-customization> | This section documents how to cope with the cases that cannot be automatically handled by the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect.  |
| <xref:inotifypropertychanged-conceptual> | This section describes the principles and concepts on which the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect relies.  |
| <xref:inotifypropertychanged-false-positives> | This section shows how to prevent notifications when the property value does not actually change. |


