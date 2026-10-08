---
uid: advisable-collections
redirect_from:
  - model/aggregatable/advisable-collections
title: "Working With Child Collections"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a comprehensive guide on working with child collections in PostSharp, focusing on advisable collections, their implementation, and their advantages over standard .NET collections. It also discusses parent surrogates, enumerating children, collections of references, and using immutable collections."
---
# Working With Child Collections

It would not be possible to implement the Aggregatable pattern without support for collection classes. However, collections of the .NET base class libraries cannot be reliably extended to support the Aggregatable pattern. Therefore, code that implements the Aggregatable pattern must rely on collection classes defined by PostSharp, namely <xref:PostSharp.Patterns.Collections.AdvisableCollection`1>, <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2>, <xref:PostSharp.Patterns.Collections.AdvisableKeyedCollection`2> and <xref:PostSharp.Patterns.Collections.AdvisableHashSet`1>. 


## Why PostSharp defines its own collection types

In the following example, an Invoice entity is composed of one instance of the `Invoice` class and several instances of the `InvoiceLine` class. The relationship between the `Invoice` and `InvoiceLine` classes is implemented using a collection. 

```csharp
[Aggregatable]
public class Invoice
{
    public Invoice()
    {
        this.Lines = new List<InvoiceLine>();
    }

    [Child]
    public IList<InvoiceLine> Lines { get; private set; }
}

[Aggregatable]
public class InvoiceLine
{    
}
```

When we add a new element to the `Lines` collection, we also need to update the parent-child relationship between the corresponding invoice and invoice line. It is not possible to do this with the standard <xref:System.Collections.Generic.List`1> class, so we need to build a specialized aggregatable collection class instead. However, we may later decide to apply another pattern to our object model, such as a threading model or undo/redo. This new pattern would, in turn, require support from the collection class. Creating new collection classes for each pattern (and potentially for each pattern combination) is clearly unmanageable. 

Instead of providing a new collection class for each specific behavior we need to inject, PostSharp introduces the concept of *advisable collections*. Advisable collections are collection classes into which PostSharp can inject behavior dynamically, at run time, according to the field to which they are assigned. Advisable collections are a way to make the collection "inherit" the pattern of the parent class. 

Let us modify our previous example to work correctly with the Aggregatable aspect.

```csharp
[Aggregatable]
public class Invoice
{
    public Invoice()
    {
        this.Lines = new AdvisableCollection<InvoiceLine>();
    }

    [Child]
    public IList<InvoiceLine> Lines { get; private set; }

}

[Aggregatable]
public class InvoiceLine
{
}
```

As you can see, the only change we made is using the <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> class instead of <xref:System.Collections.Generic.List`1>. The Aggregatable aspect applied to the `Invoice` class detects that the child property is an advisable collection and applies dynamic Aggregatable advice to the collection instance at run time. This turns our collection of invoice lines into an aggregatable collection. If we apply another aspect to the `Invoice` class later, it can add new behaviors to this collection in the same way. 


## Replacing standard collections with advisable collections

The <xref:PostSharp.Patterns.Collections> namespace defines advisable collection classes that are highly compatible with the collection types of the .NET base class libraries. 

The following table shows how advisable collections map to standard collections.

| Advisable collection | Replacement for |
|----------------------|-----------------|
| <xref:PostSharp.Patterns.Collections.AdvisableCollection`1> | <xref:System.Array>, <xref:System.Collections.Generic.List`1>, <xref:System.Collections.ObjectModel.Collection`1>, <xref:System.Collections.ObjectModel.ObservableCollection`1>  |
| <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2> | <xref:System.Collections.Generic.Dictionary`2> |
| <xref:PostSharp.Patterns.Collections.AdvisableKeyedCollection`2> | <xref:System.Collections.ObjectModel.KeyedCollection`2> |
| <xref:PostSharp.Patterns.Collections.AdvisableHashSet`1> | <xref:System.Collections.Generic.HashSet`1> |

> [!WARNING]
> <xref:PostSharp.Patterns.Collections.AdvisableDictionary`2> does not implement the <xref:System.Collections.Generic.IReadOnlyDictionary`2> interface, and <xref:PostSharp.Patterns.Collections.AdvisableHashSet`1> does not implement the <xref:System.Collections.Generic.IReadOnlyCollection`1> interface. 


## Casting advisable collections

Patterns such as Aggregatable, Recordable or Threading Models dynamically inject advices into advisable collections. These advices typically expose an interface, respectively <xref:PostSharp.Patterns.Model.IAggregatable>, <xref:PostSharp.Patterns.Recording.IRecordable> and <xref:PostSharp.Patterns.Threading.IThreadAware>. Because interfaces are introduced at run time and not at build time, you cannot use the normal type casting constructs to access the interface members. 

Instead of a normal cast, you can use the <xref:PostSharp.Patterns.DynamicAdvising.DynamicAdvisingServices.QueryInterface``1(System.Object,System.Boolean)> method to access interfaces implemented by the given instance. This method will return the proper interface implementation irrespective of how the interface is implemented: directly in the source code, introduced by PostSharp aspect at build time, or added dynamically at run time. 

The following code snippet gets the <xref:PostSharp.Patterns.Model.IAggregatable> interface of the `Lines` collection in the example above: 

```csharp
IAggregatable aggregatable = DynamicAdvisingServices.QueryInterface<IAggregatable>( invoice.Lines );
```

By default, the <xref:PostSharp.Patterns.DynamicAdvising.DynamicAdvisingServices.QueryInterface``1(System.Object,System.Boolean)> method throws `InvalidCastException` if the given instance does not implement the queried interface. You can also safely check whether the interface is implemented by passing `false` as a method argument. 

```csharp
if ( DynamicAdvisingServices.QueryInterface<IAggregatable>( collection, false ) != null )
{
}
```


## Parent surrogates

Collections play a special role in implementing the parent-child relationships between classes. Collections are often instruments instead of first-class entities of the object model. When enumerating children of a class, one generally wants to return only the items of these collections, not the collections themselves. Additionally, the Parent property of a child object should typically refer to the parent entity and not to the collection that contains the child.

Consider the following example:

```csharp
[Aggregatable]
public class Invoice
{
   public Invoice()
   {
     this.Lines = new AdvisableCollection<InvoiceLine>();
   }

   [Child]
   public IList<InvoiceLine> Lines { get; private set; }

}

[Aggregatable]
public class InvoiceLine
{
   [Parent]
   public Invoice Invoice { get; private set; }
}
```

The `Invoice` class contains a collection of `InvoiceLine` instances. We want each item of the `Lines` collection to be a child of the `Invoice` instance. However, the collection itself should not be considered a child of the `Invoice`. Additionally, we want the `InvoiceLine.Invoice` property to be set to the `Invoice`, not to the collection. 

To implement this behavior, PostSharp needs to give a different status to collections than to other entities. This concept is named a *parent surrogate*, because the collection acts as a surrogate (or proxy) between the parent and its children. 

Any aggregatable object can act as a parent surrogate, but only collections act as parent surrogates by default. You can override the default behavior by setting the <xref:PostSharp.Patterns.Model.ChildAttribute.IsParentSurrogate> property. 

In the next example, the `Lines` collection will be treated as a first-class entity. 

```csharp
[Aggregatable]
public class Invoice
{
   public Invoice()
   {
     this.Lines = new AdvisableCollection<InvoiceLine>();
   }

  [Child(IsParentSurrogate = false)]
   public IList<InvoiceLine> Lines { get; private set; }

}

[Aggregatable]
public class InvoiceLine
{
   [Parent]
   public IList<InvoiceLine> Parent { get; private set; }
}
```

To cause a custom class to behave like a parent surrogate by default, set the <xref:PostSharp.Patterns.Model.AggregatableAttribute.IsParentSurrogate> property of the <xref:PostSharp.Patterns.Model.AggregatableAttribute> applied to your class to `true`. In this case, it is not allowed to override the value in the `[Child]` attributes applied to individual properties. 

```csharp
[Aggregatable(IsParentSurrogate = true)]
```


## Enumerating children and parent surrogates

The default behavior of the <xref:PostSharp.Patterns.Model.IAggregatable.VisitChildren(PostSharp.Patterns.Model.ChildVisitor,PostSharp.Patterns.Model.ChildVisitorOptions,System.Object)> method is to skip the surrogate collection itself and invoke the <xref:PostSharp.Patterns.Model.ChildVisitor> delegate on each item of the collection. In our first example, calling the <xref:PostSharp.Patterns.Model.IAggregatable.VisitChildren(PostSharp.Patterns.Model.ChildVisitor,PostSharp.Patterns.Model.ChildVisitorOptions,System.Object)> method on the `Invoice` instance will invoke the visitor on the items of the `Lines` collection, but not on the collection instance itself. 

You can customize this behavior by providing one or more flags for the <xref:PostSharp.Patterns.Model.ChildVisitorOptions> parameter of the method. The `ChildVisitorOptions.IncludeParentSurrogates` flag will cause the visitor to be additionally invoked on the instances of the surrogate collections, while the `ChildVisitorOptions.ExcludeIndirectChildren` flag will exclude the items of such collection from being visited. 


## Collections of references

As we showed earlier, when you annotate the collection property with the `[Child]` attribute, collection items become children of the class instance. 

In certain situations, you may want to have a collection of references. The collection itself is still marked with the `[Child]` custom attribute because it would make sense from the point of view of other patterns (for instance, changes in the collection must be recorded by the Recordable pattern). However, the collection items themselves must not be considered children of the entity. 

To implement this requirement, you can set the <xref:PostSharp.Patterns.Model.ChildAttribute.ItemsRelationship> property to `RelationshipKind.Reference`. 

In the example below, the `RelatedOrders` collection is a child and therefore its changes are being recorded by the Recordable aspect. However, collection items are not children of the parent entity, because related orders do not belong to the invoice. 

```csharp
[Recordable]
public class Invoice
{
   public Invoice()
   {
     this.Lines = new AdvisableCollection<InvoiceLine>();
     this.RelatedOrders = new AdvisableCollection<Order>();
   }

   [Child]
   public IList<InvoiceLine> Lines { get; private set; }

   [Child(ItemsRelationship = RelationshipKind.Reference)]
   public IList<Order> RelatedOrders { get; private set; }

}
```


## Using immutable collections

Earlier in this article, we explained the need to replace standard .NET collections with special advisable collections of the <xref:PostSharp.Patterns.Collections> namespace. These collections come with a significant drawback: they have a significant performance and memory overhead. In many situations, collections can be replaced by immutable collections. *Immutable collections* are collections whose content never changes after instantiation. Adequate use of immutable collections can significantly improve application performance and simplify API design compared to mutable collections, whether standard or advisable. 

Immutable collections are implemented in the `System.Collections.Immutable` namespace, contained in the `System.Collections.Immutable` NuGet package. 

The Aggregatable pattern and threading models support immutable collections. When you assign an immutable collection to a child field of a parent object, items of the collection become children of the parent object. Immutable collections behave similarly to other types, so you still have to use the <xref:PostSharp.Patterns.Model.ChildAttribute> and <xref:PostSharp.Patterns.Model.ReferenceAttribute> custom attributes as usual. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Collections.AdvisableCollection`1>
<br><xref:PostSharp.Patterns.Collections.AdvisableDictionary`2>
<br><xref:PostSharp.Patterns.Collections.AdvisableKeyedCollection`2>
<br><xref:PostSharp.Patterns.Collections.AdvisableHashSet`1>
