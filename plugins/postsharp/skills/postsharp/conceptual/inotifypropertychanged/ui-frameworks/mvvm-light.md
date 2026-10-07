---
uid: inotifypropertychanged-mvvmlight
redirect_from:
  - model/notifypropertychanged/frameworks/inotifypropertychanged-mvvmlight
title: "INotifyPropertyChanged and MVVM Light Toolkit"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use PostSharp with the MVVM Light Toolkit to simplify the implementation of the INotifyPropertyChanged interface, reducing boilerplate code and potential bugs."
---
# INotifyPropertyChanged and MVVM Light Toolkit

MVVM Light is a framework that helps you build XAML applications according to the Model-View-ViewModel architectural pattern. MVVM Light includes several features, and one of those is to simplify the implementation of the <xref:System.ComponentModel.INotifyPropertyChanged> interface. However, MVVM Light still requires you to write a lot of boilerplate code. This article shows how to use PostSharp together with MVVM Light, whether because you are upgrading an existing project that already uses this framework or because you want to use the other features of MVVM Light. 


## Without PostSharp

In the example below, there is a View-Model implemented according to the MVVM Light Toolkit specification. The class `MainViewModel` inherits from the class `ViewModelBase`, which implements the <xref:System.ComponentModel.INotifyPropertyChanged> interface and thus is already helping you to use the NotifyPropertyChanged pattern. 

Part of the boilerplate that usually appears in the NotifyPropertyChanged pattern is still present. For each property, you must have an explicit field, and you must manually notify all dependencies and the property itself. The result is not only more code written, but also much room for bugs because you must discover and maintain the chain of dependent properties yourself.

```csharp
using GalaSoft.MvvmLight;
using GalaSoft.MvvmLight.Command;
using System.Windows.Input;
using System.Windows;

namespace MvvmLightTest.ViewModel
{
    public class MainViewModel : ViewModelBase
    {
        int _exampleValue;

        public int ExampleValue
        {
            get
            {
                return _exampleValue;
            }
            set
            {
                if (_exampleValue == value)
                    return;
                _exampleValue = value;
                RaisePropertyChanged("ExampleValue");
            }
        }

    }
}
```


## With PostSharp

With PostSharp, you do not need to do any of that. You indicate which classes should have the NotifyPropertyChanged pattern implemented using the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> attribute and PostSharp does the hard work for you. You can see the difference in the second example, where the previous code was refactored to keep the same functionality. 

```csharp
using GalaSoft.MvvmLight;
using GalaSoft.MvvmLight.Command;
using System.Windows.Input;
using System.Windows;
using PostSharp.Patterns.Model;

namespace MvvmLightTest.ViewModel
{
    [NotifyPropertyChanged]
    public class MainViewModel : ViewModelBase
    {
        public int ExampleValue { get; set; }
    }
}
```

Note that it is no longer necessary to derive your class from the `ViewModelBase` class. Even if you remove the inheritance from the `ViewModelBase` class, you can still use other MVVM Light features in your code such as commands. However, if you do keep the inheritance from `ViewModelBase`, the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect will invoke the `RaisePropertyChanged` method of the `ViewModelBase` class, consistently with the coding practices of MVVM Light. 


