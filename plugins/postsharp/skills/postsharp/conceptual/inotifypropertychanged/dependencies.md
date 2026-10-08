---
uid: inotifypropertychanged-dependencies
redirect_from:
  - model/notifypropertychanged/inotifypropertychanged-dependencies
title: "Implementing INotifyPropertyChanged with Properties that Depend on Other Objects"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to implement INotifyPropertyChanged with properties that depend on other objects using PostSharp. It provides a code example demonstrating the concept."
---
# Implementing INotifyPropertyChanged with Properties that Depend on Other Objects

It is very common for the properties of one class to be dependent on the properties of another class. For example, a view-model layer will often contain a reference to a model object, and public properties which are, in turn, forwarded to the underlying properties of this referenced object. In this scenario, the view-model component’s properties have a dependency on the referenced model’s properties. Subsequently, the referenced model may also have properties which depend on the properties of other objects.

PostSharp handles transitive dependencies that follow the `this.field.Property.Property` form. Add the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> class attribute to each class in the dependency chain. This will ensure that property change notifications are propagated up and down the dependency chain. PostSharp takes care of the rest and will even handle circular dependencies. 

> [!NOTE]
> Read the article <xref:inotifypropertychanged-customization> to learn about referencing more complex properties that do not follow the `this.field.Property.Property` form. 


## Example

In the following example, the `CustomerModel` class is used as a dependency of a `CustomerViewModel` class containing `FirstName` and `LastName` properties, both of which directly map to properties of the `CustomerModel` class, and a public read-only property called `FullName`, which is calculated based on the value of the underlying customer’s `FirstName` and `LastName` properties. 

```csharp
[NotifyPropertyChanged]
public class CustomerModel 
{ 
    public string FirstName { get; set; } 
    public string LastName { get; set; } 
    public string Phone { get; set; } 
    public string Mobile { get; set; } 
    public string Email { get; set; }
}

[NotifyPropertyChanged]
class CustomerViewModel
{
    CustomerModel model;

    public CustomerViewModel(CustomerModel m)
    {
        this.model = m;
    }
    
    public string FirstName { get { return this.model.FirstName; } set { this.model.FirstName = value;}}
    public string LastName { get { return this.model.LastName; } set { this.model.LastName = value; }}
    public string FullName { get { return string.Format("{0} {1}", this.model.FirstName, this.model.LastName); } }

}
```

You now have a view-model class which can be used to bridge a view (for example, an application’s user interface) with the underlying data, and calls to get/set will be propagated across the chain of dependencies.

## See Also

**Reference**

<xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>

**Other Resources**

<xref:inotifypropertychanged-customization>
