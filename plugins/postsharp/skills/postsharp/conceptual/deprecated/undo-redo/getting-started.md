---
uid: undoredo-start
redirect_from:
  - model/undoredo/undoredo-start
title: "Making Your Model Recordable"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on making an object recordable for undo/redo operations using PostSharp's RecordableAttribute aspect, and how to exclude a property from these operations."
---
# Making Your Model Recordable

> [!CAUTION]
> Undo/Redo was deprecated in PostSharp 2026.0 and will not be maintained. The API will be removed in the future.

To make an object usable for undo/redo operations, you will need to add the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect to the class. This aspect instruments changes to fields and records them into a <xref:PostSharp.Patterns.Recording.Recorder>. The aspect also instruments public methods to group field changes into logical operations. 


## Making the class recordable

1. Add the `PostSharp.Patterns.Model` package to the project. 


2. Add the <xref:PostSharp.Patterns.Recording.RecordableAttribute> to the class. 


3. Annotate your object model for parent/child relationships as explained in <xref:aggregatable-adding>. 



## Example

The following example shows an object model that has been prepared for undo/redo.

```csharp
[Recordable]
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
  
  [NotRecorded]
  public bool Persisted { get; set; }
}                

[Recordable]
public class InvoiceLine
{
  [Reference]
  private Product product;
  
  public decimal Amount { get; set; }
  
  [Parent]
  public Invoice ParentInvoice { get; private set; }
}     

[Recordable]
public class Address
{
}

[Recordable]
public class Customer
{
}
```

By adding the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect to the classes, all modifications to properties of these classes (including modifications to child collections) will be recorded. 


## Excluding a property from undo/redo

To exclude a property from participating in undo/redo, annotate that property with <xref:PostSharp.Patterns.Recording.NotRecordedAttribute>. 

```csharp
[Recordable]
public class Document
{
  public string Text { get; set; }
  
  [NotRecorded]
  public bool AnyChangesMade { get; set; }
}
```


