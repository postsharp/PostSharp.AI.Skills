---
uid: inotifypropertychanged-caliburnmicro
redirect_from:
  - model/notifypropertychanged/frameworks/inotifypropertychanged-caliburnmicro
title: "INotifyPropertyChanged and Caliburn.Micro"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses how to use PostSharp with Caliburn.Micro to simplify the implementation of the INotifyPropertyChanged interface, reducing the need for boilerplate code."
---
# INotifyPropertyChanged and Caliburn.Micro

Caliburn.Micro is a popular framework designed for building applications across all XAML platforms. Caliburn.Micro includes several features, and one of those is to simplify the implementation of the <xref:System.ComponentModel.INotifyPropertyChanged> interface. However, Caliburn.Micro still requires you to write a lot of boilerplate code. This article shows how to use PostSharp together with Caliburn.Micro, whether because you are upgrading an existing project that already uses this framework or because you want to use the other features of Caliburn.Micro. 


## Without PostSharp

The example below shows a View-Model implemented according to the Caliburn.Micro specification. The class `ShellViewModel` inherits from the class `PropertyChangedBase`, which implements the <xref:System.ComponentModel.INotifyPropertyChanged> interface and thus is already helping you to use the NotifyPropertyChanged pattern. 

Part of the boilerplate that usually appears in the NotifyPropertyChanged pattern is still present. For each property, you must have an explicit field, and you must manually notify all dependencies and the property itself. The result is not only more code written, but also much room for bugs because you must discover and maintain the chain of dependent properties yourself.

[!code-csharp[](../../../code/samples/Model/NotifyPropertyChanged/Frameworks/CaliburnMicroWithoutPostSharp.cs#example)]


## With PostSharp

With PostSharp, you do not need to do any of that. You indicate which classes should have the NotifyPropertyChanged pattern implemented using the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> attribute and PostSharp does the hard work for you. You can see the difference in the second example, where the previous code was refactored to keep the same functionality. 

[!code-csharp[](../../../code/samples/Model/NotifyPropertyChanged/Frameworks/CaliburnMicroWithPostSharp.cs#example)]

In both versions, setting the `Name` property raises the following notifications:

[!code-text[](../../../code/samples/Model/NotifyPropertyChanged/Frameworks/CaliburnMicroWithPostSharp.txt)]

Note that it is no longer necessary to derive your class from the `PropertyChangedBase` class. Even if you remove the inheritance from the `PropertyChangedBase` class, you can still use other Caliburn.Micro features in your code such as commands. However, if you do keep the inheritance from `PropertyChangedBase`, the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect will invoke the `NotifyOfPropertyChange` method of the `PropertyChangedBase` class, consistently with the coding practices of Caliburn.Micro. 


