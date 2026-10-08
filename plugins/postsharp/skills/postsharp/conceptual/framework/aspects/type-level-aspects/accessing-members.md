---
uid: members
redirect_from:
  - custompatterns/aspects/compositeaspects/members
  - framework/aspects/composite-aspects/accessing-members
title: "Accessing Members of the Target Class"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp allows developers to import a delegate of a target class method, property, or event into the aspect class. This feature helps encapsulate more design patterns using aspects. It also provides a robust and predictable way to handle situations when multiple advices try to introduce or import the same member. "
---
# Accessing Members of the Target Class

PostSharp makes it possible to import a delegate of a target class method, property or event into the aspect class, so that the aspect can invoke this member.

These mechanisms allow developers to encapsulate more design patterns using aspects.


## Importing members of the target class

Importing a member into an aspect allows this aspect to invoke the member. An aspect can import methods, properties (including indexers), fields, and events.

To import a member of the target type into the aspect class:

1. Define a field into the aspect class, of the following type:

    | Member Kind | Field Type |
    |-------------|------------|
    | Method | A typed <xref:System.Delegate>, typically one of the variants of <xref:System.Action> or <xref:System.Func`1>. The delegate signature should exactly match the signature of the imported method.  |
    | Property or field | <xref:PostSharp.Aspects.Advices.Property`1>, where the generic argument is the type of the property or field.  |
    | Collection Indexer | <xref:PostSharp.Aspects.Advices.Property`2>, where the first generic argument is the type of the property value and the second is the type of the index parameter. Indexers with more than one parameter are not supported.  |
    | Event | <xref:PostSharp.Aspects.Advices.Event`1>, where the generic argument is the type of the event delegate (for instance <xref:System.EventHandler>).  |


2. Make this field public. The field cannot be static.


3. Add the custom attribute <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> to the field. As the constructor argument, pass the name of the member to be imported. 


At run time, the field is set to a delegate of the imported member. Properties and events are imported as a set of delegates (<xref:PostSharp.Aspects.Advices.Property`1.Get>, <xref:PostSharp.Aspects.Advices.Property`1.Set>; <xref:PostSharp.Aspects.Advices.Event`1.Add>, <xref:PostSharp.Aspects.Advices.Event`1.Remove>). The aspect can invoke these delegates like any other delegate. 

The property <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> determines what happens if the member cannot be found in the target class or in its parent. By default, the field has the `null` value if it cannot be bound to a member. If the property <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> is set to `true`, a compile-time error will be emitted. 


## Interactions between several member introductions and imports

Although member introduction and import may seem simple at first sight, things become more complex when several advices try to introduce or import the same member. PostSharp handles these situations in a robust and predictable way. For this purpose, it is important to process classes, aspects and advices in a consistent order.

PostSharp enforces the following order:

* Base classes are processed first, derived classes after. Therefore, when a class is being processed, all parent classes have already been fully processed.

* Aspects targeting the same class are sorted (see <xref:aspect-dependencies>) and executed. 

* Advices of the same aspect are sorted and executed in the following order:

    * Member imports that have the property <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.Order> set to `BeforeIntroductions`. 

    * Member introductions.

    * Member imports that have the property <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.Order> set to `AfterIntroductions` (this is the default value). 


Based on this well-defined order, the advices behave as follows:

| Advice | Precondition | Behavior |
|--------|--------------|----------|
| <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> | No member, or private member defined in a parent class. | Error if <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> is `true`, ignored otherwise (by default).  |
|  | Non-virtual member defined. | Member imported. |
|  | Virtual member defined. | If <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.Order> is `BeforeIntroductions`, the overridden member is imported. This is similar to calling a method with the `base` prefix in C#. Otherwise (and by default), the member is dynamically resolved using the virtual table of the target object.  |
| <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute> | No member, or private member defined in a parent class. | Member introduced. |
|  | Non-virtual member defined in a parent class | Ignored if the property <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute.OverrideAction> is `Ignore` or `OverrideOrIgnore`, otherwise fail (by default).  |
|  | Virtual member defined in a parent class | Introduce a new `override` method if the property <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute.OverrideAction> is `OverrideOrFail` or `OverrideOrIgnore`, ignore if the property is `Ignore`, otherwise fail (by default).  |
|  | Member defined in the target class (virtual or not) | Fail by default or if the property <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute.OverrideAction> is `Fail`. <br>Otherwise:<br>* Move the previous method body to a new method so that the previous implementation can be imported by advices <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> with the property <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.Order> set to `BeforeIntroductions`.<br>* Override the method with the introduced method. |

## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute>
<br><xref:PostSharp.Aspects.Advices.ImportMemberAttribute>

**Other Resources**

<xref:aspect-dependencies>
<br><xref:advice-provider>
