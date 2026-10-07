---
uid: inotifypropertychanged-customization
redirect_from:
  - model/notifypropertychanged/inotifypropertychanged-customization
title: "Handling Corner Cases of the NotifyPropertyChanged Aspect"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide to handling corner cases of the NotifyPropertyChanged aspect in PostSharp. It covers ignoring property changes, handling complex data flows, dependencies on pure methods, and collections."
---
# Handling Corner Cases of the NotifyPropertyChanged Aspect

PostSharp includes a number of attributes for customizing the default behavior and for handling special dependencies.


## Ignoring changes to properties

Use the <xref:PostSharp.Patterns.Model.IgnoreAutoChangeNotificationAttribute> class attribute to prevent an `OnPropertyChanged` event from being invoked when setting a property. 


### Example

In this example, the `CustomerModel` class contains a `Country` property among others. To prevent a property notification from being invoked when the value of this property is set, place the <xref:PostSharp.Patterns.Model.IgnoreAutoChangeNotificationAttribute> attribute above the property. 

```csharp
[NotifyPropertyChanged]
public class CustomerModel 
{ 
    public string FirstName { get; set; } 
    public string LastName { get; set; } 
    public string Phone { get; set; } 
    public string Mobile { get; set; } 
    public string Email { get; set; } 

    [IgnoreAutoChangeNotification]
    public string Country { get; set;}
}
```


## Handling virtual calls, delegates, external methods, or complex data flows

If a property getter calls a virtual method from its class or a delegate, or references a property of another object (without using the canonical form `this.field.Property`), PostSharp will generate an error because it cannot resolve such a dependency at build time. The same limitations apply when your property getter contains complex data flows, such as loops, or calls to methods (except property getters) of other classes. 

When this happens, you can either refactor your code so that it can be automatically analyzed by PostSharp, or you can take over the responsibility for analyzing the code.


### Taking responsibility for dependency analysis

To suppress the error that PostSharp emits when it is unable to fully analyze your code, add the <xref:PostSharp.Patterns.Model.SafeForDependencyAnalysisAttribute> custom attribute to the property. 

> [!NOTE]
> By using <xref:PostSharp.Patterns.Model.SafeForDependencyAnalysisAttribute>, you are taking the responsibility that your code only has dependencies that are given either in the canonical form of `this.field.Property` or explicitly using the <xref:PostSharp.Patterns.Model.Depends.On*> construct (see below). If you are using this custom attribute but have non-canonical dependencies, some property changes may not be detected, in which case no notification will be generated. 


### Adding dependencies manually

Even when a property has the <xref:PostSharp.Patterns.Model.SafeForDependencyAnalysisAttribute> attribute, PostSharp still discovers the dependencies that are in canonical form `this.field.Property`. However, PostSharp does not discover dependencies hidden under delegate or virtual method calls. 

To explicitly add a dependency to a property, you can use the <xref:PostSharp.Patterns.Model.Depends.On*> method. The expression passed to the <xref:PostSharp.Patterns.Model.Depends.On*> must be in canonical form `this.field.Property`, that is, `Depends.On( this.field.Property )`. 

The expression can also start with another property of the same class. For example, the getter of an `Alias` property that returns `this.Child` can call `Depends.On( this.Child )`. When the type of `Alias` is the type of `Child` or one of its base types, PostSharp binds `Alias` to the fields of `Child`. A property that depends on `this.Alias.Value` is then notified in two cases: when `Value` changes in the object that `Child` returns, and when `Child` is assigned another object.


### Example

In the following example, the `CustomerModel` class contains a virtual method called `ValidateCountry` which is used by the get accessor of its `Country` property. The presence of the call to the virtual method prevents PostSharp from fully understanding the dependencies of the `Country` property. PostSharp discovers the dependency to the `_country` field but cannot analyze the implementations of the `ValidateCountry` method, and therefore emits an error. By adding the <xref:PostSharp.Patterns.Model.SafeForDependencyAnalysisAttribute> attribute to the `Country` property, you remove the error. 

Even if you remove the error, PostSharp still analyzes the `Country` property getter and finds the dependency on the `_country` field. However, it does not follow the call to the `ValidateCountry` method and does not find the dependency to the `Continent` property. That is why we have to add this dependency manually by calling the `Depends.On` method. 



```csharp
[NotifyPropertyChanged]
public class Address 
{ 
  string _country;
  public string Continent { get; set; }
 
  public virtual bool ValidateCountry(string country)
  {
     return GeoService.ContinentContains( this.Continent, country );
  }

  [SafeForDependencyAnalysis]
  public string Country 
  { 
    get
    {
      Depends.On( this.Continent );
    
      if(this.ValidateCountry(_country))
        return _country;
      else
        return "Lilliput";
    }
    
    set { _country = value; }
  }
}
```


## Handling dependencies on pure methods

Often a property will depend on a method which is solely dependent on its input parameters to produce a return value. These methods are called *pure* and do not need to be analyzed. To mark a method as pure, use the <xref:PostSharp.Patterns.Model.PureAttribute> custom attribute. 


### Example

Consider the following variation to `CustomerModel` where the `ValidPhoneNumber` property logic has been moved into a static method called `GetValidPhoneNumber()` which exists in a separate helper class called `ContactHelper`. 

Since `GetValidPhoneNumber()` is a standalone method of another class, it is not analyzed. Therefore the <xref:PostSharp.Patterns.Model.PureAttribute> attribute needs to be applied to this method to acknowledge this dependency. 

```csharp
public class ContactHelper
{
  [Pure]
  public static string GetValidPhoneNumber(string firstPhoneNumber, string secondPhoneNumber)
  {
    return firstPhoneNumber ?? secondPhoneNumber;
  }
}

[NotifyPropertyChanged]
public class CustomerModel 
{ 
  public Contact PrimaryContact {get; set;}
  public Contact SecondaryContact {get; set;}

  public string ValidPhoneNumber
  { 
    get {
      return ContactHelper.GetValidPhoneNumber(this.PrimaryContact.Phone, this.SecondaryContact.Phone);
    }
  }
}
```


## Handling dependencies on collections

When working with view models, properties often contain collections and other properties that depend on the content of these collections. The default behavior of the aspect is to react only when the collection itself is replaced. However, any change of the collection content should result in a change of a dependent property.

To facilitate this, the property containing the collection should be marked with <xref:PostSharp.Patterns.Model.AggregateAllChangesAttribute>. This causes any change of the collection (adding an item, replacing an item, removing an item, etc.) to result in notification of the change to any property that depends on it. 


### Example

In the following `Recipe` class, the `LongestStep` property gets the duration of the longest step in the recipe's `Steps` collection. 

Since the collection property is marked with <xref:PostSharp.Patterns.Model.AggregateAllChangesAttribute>, each change to this collection will result in a change of the `LongestStep` property even though the property still contains the same collection object. 

```csharp
public class RecipeStep
{  
  public TimeSpan Duration { get; }
  
  public RecipeStep(TimeSpan duration)
  {
    Duration = duration;
  }
}

[NotifyPropertyChanged]
public class Recipe 
{ 
  [AggregateAllChanges]
  public ObservableCollection<RecipeStep> Steps { get; }
  
  public Recipe()
  {
    Steps = new ObservableCollection<RecipeStep>();
  }

  [SafeForDependencyAnalysis]
  public TimeSpan LongestStep
  { 
    get 
    {      
      return this.Steps.Max(x => x.Duration);
    }
  }
}
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Model.PureAttribute>
<br><xref:PostSharp.Patterns.Model.IgnoreAutoChangeNotificationAttribute>
<br><xref:PostSharp.Patterns.Model.SafeForDependencyAnalysisAttribute>
<br><xref:PostSharp.Patterns.Model.Depends.On*>
<br><xref:PostSharp.Patterns.Model.Depends>
