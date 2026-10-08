---
uid: immutable
redirect_from:
  - threading/immutable
title: "Immutable Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use the PostSharp software to create immutable objects in code, which can be safely accessed by multiple threads. It also distinguishes between 'immutable' and 'readonly', and explains how to use constructors to set up immutable objects."
---
# Immutable Threading Model

There are times when you want certain objects in your codebase to retain their post-creation state without the possibility of it ever changing. These objects are said to be immutable. Immutable objects are useful in multithreaded applications because they can be safely accessed by several threads concurrently, without the need for locking or other synchronization. PostSharp offers the <xref:PostSharp.Patterns.Threading.ImmutableAttribute> aspect that allows you to enforce this pattern on your objects. 

Changes in an object with the <xref:PostSharp.Patterns.Threading.ImmutableAttribute> aspect will be forbidden as soon as the object constructor exits. Any further attempt to modify the object will result in an <xref:PostSharp.Patterns.Threading.ObjectReadOnlyException>. 

> [!NOTE]
> The Immutable pattern can be too strong for some common object-oriented scenarios, for instance with serializable classes. In some cases, the Freezable object is a better choice. For details, see <xref:freezable>. 


## Making a class immutable


To add the Immutable pattern manually:

1. Add a reference to the `PostSharp.Patterns.Threading` package to your project. 


2. Add the <xref:PostSharp.Patterns.Threading.ImmutableAttribute> custom attribute to your class. 


3. Annotate your object model for parent/child relationships as described in <xref:aggregatable-adding>. 



## Rules enforced by the Immutable aspect

An immutable object will throw the following exceptions at run time:

* <xref:PostSharp.Patterns.Threading.ThreadMismatchException> if both of the following conditions are simultaneously true: 

    * you access the object from a different thread than the one that created it, and

    * the constructor has not completed yet.

* <xref:PostSharp.Patterns.Threading.ObjectReadOnlyException> if a field or property is being modified after the constructor has completed. 


## Immutable object trees

Because the Immutable pattern is an implementation of the Aggregatable pattern, all of the same behaviors of the <xref:PostSharp.Patterns.Model.AggregatableAttribute> are available. As a result, you can create both immutable classes and immutable object trees. For more information regarding object trees, read <xref:aggregatable>. 

> [!NOTE]
> Children of immutable objects must be marked as Immutable or Freezable themselves. Adding <xref:PostSharp.Patterns.Threading.ImmutableAttribute> or <xref:PostSharp.Patterns.Threading.FreezableAttribute> to the child classes will accomplish this. Freezable children will be automatically frozen when the constructor of the parent completes. 


## Immutable vs readonly

Many C# developers make use of the `readonly` keyword in an attempt to make their objects immutable. The `readonly` keyword does not guarantee immutability. Using `readonly` only ensures that no method other than the object's constructor can alter the variable's value. It does not, however, prevent you from altering values on complex objects outside of the constructor. 


### The readonly keyword may be too strict

In the following code sample, the `_id` variable is a primitive type and cannot be altered outside the constructor. This is enforced at compile time and an error would be displayed where the `SetIdentifier` method attempts to change the `_id` field value. The compiler does not see that the `SetIdentifier` method is called only from the constructor. In this example, the `readonly` keyword is too strong even if the class is legitimately immutable. 

```csharp
public class Invoice
{
  public readonly long _id;
  
  public Invoice(long id)
  {
    SetIdentifier(id);
  }
  
  private void SetIdentifier(long id)
  {
    // Will cause compilation error.
    _id = id;
  }
}
```


### The readonly keyword may also be too loose

When you have complex entities composed of several objects, immutability is a characteristic of the whole entity, not of a single object. However, this does not fit with the `readonly` keyword. 

In the following example, the `Invoice` is not immutable even if the `_invoiceHeader` field is `readonly`. 

```csharp
public class Invoice
{
  public readonly InvoiceHeader _invoiceHeader;
  
  public Invoice()
  {
    _invoiceHeader = new InvoiceHeader();
  }
  
  public void Refresh()
  {
    // Valid but not immutable.
    _invoiceHeader.CustomerName = "Jim";
    _invoiceHeader.CustomerPhone = "555-123-9876";
  }
}
```

The same type of change to object state can happen with collections. You cannot reinitialize a `readonly` collection, but you can freely `Add`, `Remove`, `Clear` and do other operations that the collection itself exposes. Additionally, if the collection contains complex types, you are able to change values on each instance that the collection contains. 

```csharp
public class Invoice
{
  public readonly IList<Item> _items;
  public Invoice()
  {
    _items = new List<Item>();
  }
  
  public void Refresh()
  {
    //will cause a compilation error
    _items = new List<Item>();
    
    //valid but not immutable
    _items.Add(new Item());
    _items[0].Price = 3.50;
    _items.RemoveAt(0);
  }
}
```

As you can see, there is no way to use the `readonly` keyword to make complex object graphs immutable. Combining the <xref:PostSharp.Patterns.Threading.ImmutableAttribute>, <xref:PostSharp.Patterns.Model.ChildAttribute>, <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> and the <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2> types allows you to make immutable objects that guarantee no changes to primitive or complex objects after constructor execution has completed. 


## Constructor execution

Objects are not frozen until the last constructor has finished executing. Because of this, you can use the constructor to set up the state of the parent instance through its own constructor as well as chained, or inherited object, constructors. You are also able to make changes to child object instances through their constructors at this time.

```csharp
[Immutable]
public class Invoice : Document
{
  public Invoice(long id) : base(id)
  {
    Items = new AdvisableCollection<Item>();
    Items.Add(new Item("widget"));
  }
  
  [Child]
  public AdvisableCollection<Item> Items { get; set; }
}

[Immutable]
public class Document
{
  private long _id;
  public Document (long id)
  {
    _id = id;
  }
}

[Immutable]
public class Item
{
  public Item (string name)
  {
    Name = name;
  }
  public string Name { get; set; }
}
```

In this example, the constructors finish executing in the order of `Document`, `Item` and finally `Invoice`. It is not until after the `Invoice` constructor finishes executing that the object graph is made immutable. 


## Immutable collections

When authoring immutable object models, immutable collections are a good replacement for advisable collections. Immutable collections are implemented in the `System.Collections.Immutable` namespace, contained in the `System.Collections.Immutable` NuGet package. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Threading.ImmutableAttribute>
<br><xref:PostSharp.Patterns.Model.AggregatableAttribute>
<br><xref:PostSharp.Patterns.Collections.AdvisableDictionary`2>
<br><xref:PostSharp.Patterns.Collections.AdvisableCollection`1>

**Other Resources**

<xref:immutable-collections>
