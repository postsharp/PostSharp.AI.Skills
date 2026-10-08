---
uid: aggregatable-adding
redirect_from:
  - model/aggregatable/aggregatable-adding
title: "Annotating an Object Model for Parent/Child Relationships (Aggregatable)"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on using PostSharp to describe parent-child relationships in an object model, including steps to apply the Aggregatable aspect and custom attributes to classes, fields, and properties."
---
# Annotating an Object Model for Parent/Child Relationships (Aggregatable)

PostSharp provides several custom attributes that you can apply to your object model to describe the parent-child relationships in a natural and concise way. The <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect is applied to the object model classes, and the properties are marked with <xref:PostSharp.Patterns.Model.ChildAttribute>, <xref:PostSharp.Patterns.Model.ReferenceAttribute> and <xref:PostSharp.Patterns.Model.ParentAttribute> custom attributes. You can also use <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> and <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2> classes to make your collection properties aware of the Aggregatable pattern. 

The following procedure shows how to add parent-child relationships to existing object models.


## Applying the Aggregatable pattern to an object model

1. Add the `PostSharp.Patterns.Aggregation` package to your project. 


2. Add the <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect to all base classes. Note that the aspect is inherited, so it is not necessary to explicitly add the aspect to classes that derive from a class that already has the aspect. Note also that the <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect is implicitly added by other aspects, including all threading models (<xref:PostSharp.Patterns.Threading.ThreadAwareAttribute>), <xref:PostSharp.Patterns.Model.DisposableAttribute> and <xref:PostSharp.Patterns.Recording.RecordableAttribute>. 

    > [!NOTE]
    > It is not strictly necessary to add the <xref:PostSharp.Patterns.Model.AggregatableAttribute> aspect to a class whose instances will be children but not parents unless you want to track the relationship to the parent using the <xref:PostSharp.Patterns.Model.IAggregatable.Parent> property or the <xref:PostSharp.Patterns.Model.ParentAttribute> custom attribute in this class (see below). 


3. Annotate fields and automatic properties of all aggregatable classes with the <xref:PostSharp.Patterns.Model.ChildAttribute> or <xref:PostSharp.Patterns.Model.ReferenceAttribute> custom attribute. Fields or properties of a value type must not be annotated. 


4. Collections require special attention:

    * Modify the code to use <xref:PostSharp.Patterns.Collections.AdvisableCollection`1>, <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2> or <xref:PostSharp.Patterns.Collections.AdvisableKeyedCollection`2> instead of standard .NET collections for child fields. This change is necessary because all objects assigned to child fields/properties must be aware of the Aggregatable pattern. 

    * By default, the items of a child collection are children of the object that owns the collection. If the collection itself is a child but its items should be considered as references, set the <xref:PostSharp.Patterns.Model.ChildAttribute.ItemsRelationship> property to `RelationshipKind.Reference`. 

    See <xref:advisable-collections> for details. 


5. Optionally, add a field or property to link back from the child object to the parent, and add the <xref:PostSharp.Patterns.Model.ParentAttribute> to this field/property. PostSharp will automatically update this field or property to make sure it refers to the parent object. 

    > [!TIP]
    > For better encapsulation, setters of parent properties should have `private` visibility. In case of parent fields, the `private` visibility is preferred. User code should not manually set a parent field or property. 



## Example

In the following example, an `Invoice` object owns several instances of the `InvoiceLine` class, therefore both classes must be annotated with <xref:PostSharp.Patterns.Model.AggregatableAttribute>. However, the `Invoice` does not own the `Customer` to which it is associated, so the `Customer` class does not need the custom attribute. 

Note that in the constructor of the `Invoice` class, we assign an <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> to the `Lines` field instead of a `List`. 

```csharp
[Aggregatable]
public class Invoice
{
  public Invoice()
  {
     this.Lines = new AdvisableCollection<InvoiceLine>();
  }

  [Reference]
  public Customer Customer { get; set; }
  
  [Child]
  public IList<InvoiceLine> Lines { get; private set; }
  
  [Child]
  public Address DeliveryAddress { get; set; }
}                

[Aggregatable]
public class InvoiceLine
{
  [Reference]
  private Product product;
  
  public decimal Amount { get; set; }
  
  [Parent]
  public Invoice ParentInvoice { get; private set; }
}     

[Aggregatable]
public class Address
{
}

public class Customer
{
}
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Model.AggregatableAttribute>
<br><xref:PostSharp.Patterns.Model.ParentAttribute>
<br><xref:PostSharp.Patterns.Model.ChildAttribute>
<br><xref:PostSharp.Patterns.Model.ReferenceAttribute>
<br><xref:PostSharp.Patterns.Collections.AdvisableCollection`1>
<br><xref:PostSharp.Patterns.Collections.AdvisableDictionary`2>

**Other Resources**

<xref:aggregatable-adding-programmatically>
