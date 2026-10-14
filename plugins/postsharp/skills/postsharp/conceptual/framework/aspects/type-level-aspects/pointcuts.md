---
uid: pointcuts
title: "Selecting Members with Pointcuts"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how the advices of a type-level or assembly-level aspect select the members they transform with pointcuts, how to select members with code, and how advices are grouped into one transformation."
---
# Selecting Members with Pointcuts

In a type-level or assembly-level aspect, an advice usually transforms members of the target, not the target itself. For instance, an aspect applied to a class intercepts the setters of its properties. A *pointcut* is a custom attribute on the advice method that selects these members.

A pointcut should select only declarations that are inside the target of the aspect. If an aspect is applied to a class `A`, the pointcut can select the class `A` and the members of `A`, but not other classes or their members. PostSharp enforces this rule, with the error LA0110, only for instance-scoped aspects, such as an <xref:PostSharp.Aspects.InstanceLevelAspect>. An aspect is instance-scoped when it implements <xref:PostSharp.Aspects.IInstanceScopedAspect> and is applied to a type or to an instance member. A <xref:PostSharp.Aspects.Advices.MethodPointcut> of another aspect can select a member of any type of the current assembly without error.

In an assembly-level aspect, <xref:PostSharp.Aspects.Advices.MulticastPointcut> selects nothing, and <xref:PostSharp.Aspects.Advices.MatchPointcut> causes the error LA0251. Use a <xref:PostSharp.Aspects.Advices.MethodPointcut> whose parameter is an <xref:System.Reflection.Assembly>, or add type-level aspects with <xref:PostSharp.Aspects.IAspectProvider> (see <xref:aspect-provider>).

In a member-level aspect, such as a <xref:PostSharp.Aspects.MethodLevelAspect>, you do not need a pointcut: an advice without pointcut applies to the target of the aspect (see <xref:member-level-aspects>).


## Multicast pointcut

The pointcut type <xref:PostSharp.Aspects.Advices.MulticastPointcut> selects members declaratively, with the properties of a single custom attribute. It works like <xref:PostSharp.Extensibility.MulticastAttribute> (see <xref:multicast>): you filter the kind of declarations, their names and their attributes.

The following code applies the `OnPropertySet` advice to all non-abstract instance properties of the class to which the aspect is applied. The advice is generic: for each property, `T` is the type of the property, so the value is not boxed. The advice stores the value through the binding of the property.

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMulticastPointcut.cs#advice)]

When this advice is part of a type-level aspect applied to a `Customer` class, setting the `Name` and `Address` instance properties and the static `Count` property prints the following output:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMulticastPointcut.txt)]


## Method pointcut

The pointcut type <xref:PostSharp.Aspects.Advices.MethodPointcut> selects members with a method of the aspect class. The argument of the custom attribute is the name of this method. Use the `nameof` operator, so that the compiler checks the name.

The pointcut method is an instance method of the aspect class, and it can be private. Its only parameter is of type <xref:System.Object> or of the reflection type of the declaration to which the *aspect* applies, such as <xref:System.Type> or <xref:System.Reflection.Assembly>. It returns an <xref:System.Collections.Generic.IEnumerable`1> of the reflection type of the declarations to which the *advice* applies, such as <xref:System.Reflection.PropertyInfo>. Otherwise, PostSharp reports the error LA0101 or LA0102.

The following code applies the `OnPropertySet` advice to all writable properties that are not annotated with the `IgnorePropertyChanged` custom attribute. 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMethodPointcut.cs#pointcut)]

When the `Address` property of the target class is annotated with `[IgnorePropertyChanged]`, setting the `Name` and `Address` properties prints the following output:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMethodPointcut.txt)]

Pointcut methods run at build time and can use LINQ to query <xref:System.Reflection>.


## Match pointcut

By default, an advice that does not fit a target causes an error. For instance, `[Argument( 1 )]` causes the error LA0225 on a method that has only one parameter. The <xref:PostSharp.Aspects.Advices.MatchPointcut> pointcut skips these targets instead. It requires an aspect applied to a type, such as a <xref:PostSharp.Aspects.TypeLevelAspect>. On any other target, it causes the error LA0251. It selects the methods declared by the target type that have a given name, including static and non-public methods, but not inherited methods. Each advice applies only to the methods whose signature matches its bound parameters (see <xref:advice-parameter-binding>). Several advices can therefore handle different overloads of the same method.

When <xref:PostSharp.Aspects.Advices.MatchPointcut.MatchParameterCount> is `true`, which is the default value, the pointcut also requires that the number of parameters of the method is the highest index of the `[Argument( index )]` parameters of the advice, plus one.

Only the `[Argument]` parameters that select the argument by its index are counted. When the advice has none, all the methods with the name are selected.

The following aspect validates the arguments of the overloads of the `Catalog.Add` method. The first advice applies to `Add( string )`, and the second one to `Add( string, string )`. The method `Add( int )` matches none of them, so it is left unchanged. The `nameof` operator names the method, so that the compiler checks the name.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutMatch.cs#example)]

When a program calls `Add( "" )`, `Add( "", "value" )` and `Add( 0 )`, it prints the following output:

[!code-text[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutMatch.txt)]

A target is skipped when it does not have the signature of the advice, for instance when an argument is missing or has an incompatible type, or when two bound values give conflicting type arguments. An error that does not come from the signature, such as a violated generic constraint, is still reported.


## Self pointcut

The pointcut type <xref:PostSharp.Aspects.Advices.SelfPointcut> selects the target of the aspect. You rarely need it: an advice without pointcut already applies to the target of the aspect when it accepts this kind of declaration. When it does not, for instance a method boundary advice in a <xref:PostSharp.Aspects.TypeLevelAspect>, PostSharp reports the error LA0049, and the advice needs another pointcut.

<xref:PostSharp.Aspects.Advices.SelfPointcut> makes an advice an explicit master advice. You need it only to declare several groups of the same family on the target of the aspect: give each master advice a <xref:PostSharp.Aspects.Advices.SelfPointcut>, and set the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property of the other advices (see [Grouping advices](#grouping-advices)). Existing aspects that use <xref:PostSharp.Aspects.Advices.SelfPointcut> where it is not needed keep working.


## Selecting targets programmatically

Pointcuts select the targets of the advices declared in the aspect class. PostSharp provides three ways to select targets with code instead of custom attributes:

* <xref:PostSharp.Aspects.Advices.MethodPointcut> selects the targets of a declared advice with a method of the aspect, as shown above. Use it when the advice is known in advance and only its targets depend on the code.
* <xref:PostSharp.Aspects.Advices.IAdviceProvider> provides, for each target of the aspect, advices that import or introduce members, such as an import of several fields of the target class into one aspect field. Use it when the advices themselves depend on the target. See <xref:advice-provider>.
* <xref:PostSharp.Aspects.IAspectProvider> adds other aspects to declarations of the target. Use it when you need whole aspects, possibly of different types, on members that you select with code. See <xref:aspect-provider>.


## Grouping advices

Advices of different kinds but of the same family can be grouped, so that they are woven as one transformation. For instance, the method boundary advices of a group are woven as one `try` / `catch` / `finally` construct, as <xref:PostSharp.Aspects.OnMethodBoundaryAspect> is.


### Why group advices

Consider three advices of the method boundary family: <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice> and <xref:PostSharp.Aspects.Advices.OnMethodExceptionAdvice>. The order of these advices is important, because it results in different generated code for the `try` / `catch` / `finally` block. 

The following two code blocks compare two results. In the first block, the advices are separate transformations, ordered as `OnEntry`, `OnExit`, `OnException`. In the second block, the advices are grouped. 

<!-- not-compiled: equivalent of the code generated by PostSharp -->
```csharp
void Method()
{
  try
  {
    OnEntry();

    try
    {
      // Original method body.
    }
    finally
    {
      OnExit();
    }
  }
  catch
  {
     OnException();
     throw;
  }
}
```

<!-- not-compiled: equivalent of the code generated by PostSharp -->
```csharp
void Method()
{
  OnEntry();

  try
  {
      // Original method body.
  }
  catch
  {
     OnException();
     throw;
  }
  finally
  {
      OnExit();
  }
}
```

The code in the first block can make sense in some situations, but it is not consistent with the code generated by <xref:PostSharp.Aspects.OnMethodBoundaryAspect>. With the order `OnEntry`, `OnException`, `OnExit`, separate advices would generate the same code as the second block, but you would have to specify the order with custom attributes (see <xref:advice-ordering>). Grouping is the simpler way to get a consistent result.

The advices of a group can also share variables during one execution of the target method, through parameters annotated with <xref:PostSharp.Aspects.Advices.StateAttribute> (see [How advices are grouped](#how-advices-are-grouped)). The reasons to group the location interception advices and the event interception advices are similar: the advices of a group behave as a single transformation (see <xref:simple-aspects>). 


### How advices are grouped

A group has one *master advice*. The other advices of the group are *subordinate advices*. Only the master advice can have a pointcut, and the subordinate advices apply to the declarations that this pointcut selects.

In most aspects, PostSharp infers the groups. An advice without pointcut and without the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property joins the group of the only master advice of the same family declared in its class, or else in the nearest base class that declares one. An advice that another advice names in its <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property is a master advice, even without pointcut; it then applies to the target of the aspect.

The following code groups an <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice> and an <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice>. `OnEntry` is the master advice because it has a pointcut, and `OnExit` joins its group. The `stopwatch` parameters of both advices are bound to the same variable: the entry advice assigns it, and the exit advice reads it. 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceGrouping.cs#advices)]


### Several groups of the same family

When the class declares several master advices of the same family, PostSharp cannot infer the group of an advice without pointcut, and reports the error LA0230. Set the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property of each subordinate advice to the name of its master advice method, with the `nameof` operator.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutSeveralGroups.cs#advices)]

PostSharp reports the following errors and warnings about groups:

| Code | Severity | Meaning |
|------|----------|---------|
| LA0049 | Error | An advice without pointcut cannot apply to the target of the aspect. Add a pointcut. |
| LA0059 | Error | A group contains two advices of the same kind, for instance two entry advices. |
| LA0230 | Error | The group of an advice cannot be inferred because several master advices are candidates. Set the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property. |
| LA0135 | Error | An advice that sets <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> also sets <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> or <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.UnsupportedTargetAction>, which only a master advice can set. |
| LA0231 | Error | PostSharp infers a group, and several of its advices set <xref:PostSharp.Aspects.Advices.Advice.Description>, <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> or <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.UnsupportedTargetAction>, so the master advice cannot be selected. |
| LA0232 | Warning | A subordinate advice has a pointcut, which is ignored. |


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.MulticastPointcut>
<br><xref:PostSharp.Aspects.Advices.MethodPointcut>
<br><xref:PostSharp.Aspects.Advices.MatchPointcut>
<br><xref:PostSharp.Aspects.Advices.SelfPointcut>
<br><xref:PostSharp.Aspects.Advices.GroupingAdvice.Master>

**Other Resources**

<xref:complex-aspects>
<br><xref:advices>
<br><xref:advice-parameter-binding>
<br><xref:advice-provider>
<br><xref:aspect-provider>
<br><xref:advice-ordering>
