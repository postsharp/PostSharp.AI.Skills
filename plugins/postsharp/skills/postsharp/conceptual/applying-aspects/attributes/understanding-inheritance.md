---
uid: multicast-inheritance
redirect_from:
  - custompatterns/aspects/applying/multicast-inheritance
title: "Understanding Aspect Inheritance"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains aspect inheritance in PostSharp, including the difference between strict and multicast inheritance. It also discusses how to avoid duplicate aspects."
---
# Understanding Aspect Inheritance


## Lines of Inheritance

Aspect inheritance is supported on the following elements:

| Aspect applied on | Aspect propagated to |
|-----------------------------------------------|--------------------------------------------------|
| Interface | Any class implementing this interface or any other interface deriving this interface. |
| Class | Any class derived from this class. |
| Virtual or Abstract Methods | Any method implementing or overriding this method. |
| Interface Methods | Any method implementing that interface semantic. |
| Parameter or Return Value of an abstract, virtual or interface method | The corresponding parameter or the return value of derived methods using the method-level rules described above. |
| Assembly | All assemblies referencing (directly or not) this assembly. |

> [!NOTE]
> Aspect inheritance is not supported on events and properties, but it is supported on event and property accessors. The reason for this limitation is that there is actually nothing like *event inheritance* or *property inheritance* in MSIL (events and properties have nearly no existence for the CLR: they are pure metadata intended for compilers). Aspect inheritance is also not supported on fields. 


## Strict and Multicast Inheritance

To understand the difference between strict and multicast inheritance, remember the original role of <xref:PostSharp.Extensibility.MulticastAttribute>: to propagate custom attributes along the lines of containment. So, if you apply a method-level attribute to a type, the attribute will be propagated to all the methods of this type (some methods can be filtered out using specific properties of <xref:PostSharp.Extensibility.MulticastAttribute>, or <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute>; see <xref:multicast> for details). 

The difference between strict and multicast inheritance is that, with multicast inheritance (but not with strict inheritance), even inherited attributes are propagated along the lines of containment.

Consider the following piece of code, where `A` and `B` are both method-level aspects. 

```csharp
[A(AttributeInheritance = MulticastInheritance.Strict)]
          [B(AttributeInheritance = MulticastInheritance.Multicast)]
          public class BaseClass
          {
            // Aspect A, B.
            public virtual void Method1() {}
          }

          public class DerivedClass : BaseClass
          {
            // Aspects A, B.
            public override void Method1() {}

            // Aspect B.
            public void Method2() {}
          }
```

If you look only at `BaseClass`, there is no difference between strict and multicast inheritance. However, if you look at `DerivedClass`, you see the difference: only aspect `B` is applied to `Method2`. 

The multicasting mechanism for aspect `A` is the following: 

* Propagation along the lines of containment from `BaseClass` to `BaseClass.Method1`. 

* Propagation along the lines of inheritance from `BaseClass.Method1` to `DerivedClass.Method1`. 

For aspect `B`, the mechanism is the following: 

* Propagation along the lines of containment from `BaseClass` to `BaseClass.Method1`. 

* Propagation along the lines of inheritance from `BaseClass.Method1` to `DerivedClass.Method1`. 

* Propagation along the lines of inheritance from `BaseClass` to `DerivedClass`. 

* Propagation along the lines of containment from `DerivedClass` to `DerivedClass.Method1` and `DerivedClass.Method2`. 

In other words, the difference between strict and multicast inheritance is that multicast inheritance applies containment propagation rules to inherited aspects; strict inheritance does not.


### Avoiding Duplicate Aspects

If you read the multicasting mechanism for aspect `B` again, you will notice that aspect `B` is actually applied twice to `DerivedClass.Method1`: one instance comes from the inheritance propagation from `BaseClass.Method1`, the other instance comes from containment propagation from `DerivedClass`. 

To avoid surprises, PostSharp implements a mechanism to avoid duplicate aspect instances. The rule: if many paths lead from the same custom attribute usage to the same target element, only one instance of this custom attribute is applied to the target element.

> [!CAUTION]
> You can still have many instances of the same custom attribute on the same target element if they have different origins (that is, they typically originate from different lines of code). You can enforce uniqueness of custom attribute instances by using <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowMultiple>. See the section <xref:multicast-override> for details. 

## See Also

**Reference**

<xref:PostSharp.Extensibility.MulticastAttribute>
<br><xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute>
