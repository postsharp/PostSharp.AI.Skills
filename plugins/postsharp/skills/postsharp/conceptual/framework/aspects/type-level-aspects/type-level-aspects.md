---
uid: complex-aspects
redirect_from:
  - custompatterns/aspects/compositeaspects/complex-aspects
  - framework/aspects/composite-aspects/composite-aspects
title: "Developing Type- and Assembly-Level Aspects"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how to develop aspects that apply to a type or an assembly and transform several of its members with advices and pointcuts, introduce and import members, and add other aspects or advices with code."
---
# Developing Type- and Assembly-Level Aspects

A *type-level aspect* applies to a type, and an *assembly-level aspect* applies to an assembly. One such aspect can transform several members of its target, introduce new members into the target type, and access the existing members of the target type. This kind of aspect implements design patterns that span a whole class, such as `INotifyPropertyChanged` or a threading model.

The aspect derives from one of the following classes:

| Base class | Target | Aspect instances |
|------------|--------|------------------|
| <xref:PostSharp.Aspects.TypeLevelAspect> | A class, a struct or an interface | One instance per target type. |
| <xref:PostSharp.Aspects.InstanceLevelAspect> | A class that is not static | One instance per instance of the target class, so the aspect can have a state for each object (see <xref:aspect-lifetime>). Introductions and imports of members require this base class. |
| <xref:PostSharp.Aspects.AssemblyLevelAspect> | An assembly | One instance per assembly. Its advices need a <xref:PostSharp.Aspects.Advices.MethodPointcut>, and it cannot introduce or import members. |

The aspect class contains:

* *advice methods*, which add behaviors to members of the target. They are written as in member-level aspects (see <xref:advices>), and their parameters receive typed values of the target (see <xref:advice-parameter-binding>).
* *pointcuts*, custom attributes on the advice methods that select the members to which each advice applies (see <xref:pointcuts>).
* members that the aspect introduces into the target type, or that the aspect imports from it (see <xref:code-injections> and <xref:members>).

To add behaviors to a single method, field, property or event, a member-level aspect is simpler (see <xref:member-level-aspects>).


## Example

The following aspect writes a line to the console whenever an instance property of the target class is set. The pointcut selects the properties, and the advice is generic, so the value is not boxed. Apply it to classes only: on a struct, the `instance` parameter receives a boxed copy, so the value would be stored in the copy.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/TraceProperties.cs#aspect)]

When the aspect is applied to a `Customer` class, setting the `Name` and `Age` instance properties and a static property prints the following output:

[!code-text[](../../../../code/samples/framework/aspects/type-level-aspects/TraceProperties.txt)]


## In this chapter

| Section | Description |
|---------|-------------|
| <xref:pointcuts> | This article explains how pointcuts select the members to which advices apply, and how advices are grouped into one transformation. |
| <xref:code-injections> | This article shows how to introduce an interface, method, property or event into the target class. The new interface or member is implemented by the aspect itself. |
| <xref:members> | This article shows how to import a member of the target class so that the aspect can invoke it through a delegate. It also explains how member introductions and imports interact. |
| <xref:aspect-provider> | This article shows how an aspect can add other aspects to declarations that it selects with code. |
| <xref:advice-provider> | This article shows how an aspect can provide import and introduction advices for each target with code. |


## See Also

**Reference**

<xref:PostSharp.Aspects.TypeLevelAspect>
<br><xref:PostSharp.Aspects.InstanceLevelAspect>
<br><xref:PostSharp.Aspects.AssemblyLevelAspect>
<br><xref:PostSharp.Aspects.Advices>

**Other Resources**

<xref:member-level-aspects>
<br><xref:simple-aspects>
