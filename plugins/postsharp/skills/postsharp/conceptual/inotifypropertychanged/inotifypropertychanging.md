---
uid: inotifypropertychanging
redirect_from:
  - model/notifypropertychanged/inotifypropertychanging
title: "Implementing INotifyPropertyChanging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions on implementing the INotifyPropertyChanging interface with PostSharp, which signals before a property value changes, by creating an 'OnPropertyChanging' method."
---
# Implementing INotifyPropertyChanging

By convention, the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event must be raised *after* the property value has changed. However, some components need to be signaled *before* the property value is changed. This is the role of the <xref:System.ComponentModel.INotifyPropertyChanging> interface. 

Because the <xref:System.ComponentModel.INotifyPropertyChanging> interface is not portable (in Xamarin, it is even a part of a different namespace), PostSharp cannot introduce it. However, if you implement the <xref:System.ComponentModel.INotifyPropertyChanging> interface yourself in your code, PostSharp will signal the <xref:System.ComponentModel.INotifyPropertyChanging.PropertyChanging> event. To make that work, you need to create an `OnPropertyChanging` method with the right signature. 


## Adding the INotifyPropertyChanging interface to a class

1. Make your class implement <xref:System.ComponentModel.INotifyPropertyChanging> and add the <xref:System.ComponentModel.INotifyPropertyChanging.PropertyChanging> event. 


2. Add the `OnPropertyChanging` method with exactly the following signature, and invoke the <xref:System.ComponentModel.INotifyPropertyChanging.PropertyChanging> event. 

    ```csharp
    protected void OnPropertyChanging( string propertyName )
    {
        if ( this.PropertyChanging != null )
        {
          this.PropertyChanging( this, new PropertyChangingEventArgs ( propertyName ) );
        }
    }
    ```


3. Add the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect to your class as described in <xref:inotifypropertychanged-add>. 


> [!NOTE]
> The contract between your class and the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> is only the `OnPropertyChanging` method. As long as this method exists in the class, it will be invoked by the aspect before the value of a property changes. 


## Example

The following example demonstrates how to implement <xref:System.ComponentModel.INotifyPropertyChanging>. 

```csharp
[NotifyPropertyChanged]                
public class MyClass : INotifyPropertyChanging
{
    public event PropertyChangingEventHandler PropertyChanging;
    
    protected void OnPropertyChanging( string propertyName )
    {
       if ( this.PropertyChanging != null )
       {
          this.PropertyChanging( this, new PropertyChangingEventArgs ( propertyName ) );
       }
    }
    
    public string MyProperty { get; set; }
}
```


