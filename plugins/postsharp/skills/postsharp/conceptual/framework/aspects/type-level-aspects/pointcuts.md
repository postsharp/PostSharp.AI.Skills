---
uid: pointcuts
title: "Selecting Members with Pointcuts"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how the advices of a type-level or assembly-level aspect select the members they transform with pointcuts, how to select members with code, and how advices are grouped into one transformation."
---
# Selecting Members with Pointcuts

In a type-level or assembly-level aspect, an advice usually transforms members of the target, not the target itself. For instance, an aspect applied to a class intercepts the setters of its properties. A *pointcut* is a custom attribute on the advice method that selects these members.

A pointcut should select only declarations that are inside the target of the aspect. If an aspect is applied to a class `A`, the pointcut can select the class `A` and the members of `A`, but not other classes or their members. PostSharp enforces this rule, with the error <xref:la0110?text=LA0110>, only for instance-scoped aspects, such as an <xref:PostSharp.Aspects.InstanceLevelAspect>. An aspect is instance-scoped when it implements <xref:PostSharp.Aspects.IInstanceScopedAspect> and is applied to a type or to an instance member. A <xref:PostSharp.Aspects.Advices.MethodPointcut> of another aspect can select a member of any type of the current assembly without error.

In an assembly-level aspect, <xref:PostSharp.Aspects.Advices.MulticastPointcut> selects nothing, and <xref:PostSharp.Aspects.Advices.MatchPointcut> causes the error <xref:la0251?text=LA0251>. Use a <xref:PostSharp.Aspects.Advices.MethodPointcut> whose parameter is an <xref:System.Reflection.Assembly>, or add type-level aspects with <xref:PostSharp.Aspects.IAspectProvider> (see <xref:aspect-provider>).

In a member-level aspect, such as a <xref:PostSharp.Aspects.MethodLevelAspect>, you do not need a pointcut: an advice without pointcut applies to the target of the aspect (see <xref:member-level-aspects>).


## Multicast pointcut

The pointcut type <xref:PostSharp.Aspects.Advices.MulticastPointcut> selects members declaratively, with the properties of a single custom attribute. It works like <xref:PostSharp.Extensibility.MulticastAttribute> (see <xref:multicast>): you filter the kind of declarations, their names and their attributes.

The following code applies the `OnPropertySet` advice to all non-abstract instance properties of the class to which the aspect is applied. The advice is generic: for each property, `T` is the type of the property, so the value is not boxed. The advice stores the value through the binding of the property.

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMulticastPointcut.cs#advice)]

When this advice is part of a type-level aspect applied to a `Customer` class, setting the `Name` and `Address` instance properties and the static `Count` property prints the following output:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMulticastPointcut.txt)]


## Method pointcut

The pointcut type <xref:PostSharp.Aspects.Advices.MethodPointcut> selects members with a method of the aspect class. The argument of the custom attribute is the name of this method. Use the `nameof` operator, so that the compiler checks the name.

The pointcut method is an instance method of the aspect class, and it can be private. Its only parameter is of type <xref:System.Object> or of the reflection type of the declaration to which the *aspect* applies, such as <xref:System.Type> or <xref:System.Reflection.Assembly>. It returns an <xref:System.Collections.Generic.IEnumerable`1> of the reflection type of the declarations to which the *advice* applies, such as <xref:System.Reflection.PropertyInfo>. Otherwise, PostSharp reports the error <xref:la0101?text=LA0101> or <xref:la0102?text=LA0102>.

The following code applies the `OnPropertySet` advice to all writable properties that are not annotated with the `IgnorePropertyChanged` custom attribute. 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMethodPointcut.cs#pointcut)]

When the `Address` property of the target class is annotated with `[IgnorePropertyChanged]`, setting the `Name` and `Address` properties prints the following output:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceMethodPointcut.txt)]

Pointcut methods run at build time and can use LINQ to query <xref:System.Reflection>.


## Match pointcut

By default, an advice that does not fit a target causes an error. For instance, `[Argument( 1 )]` causes the error <xref:la0225?text=LA0225> on a method that has only one parameter. The <xref:PostSharp.Aspects.Advices.MatchPointcut> pointcut skips these targets instead. It requires an aspect applied to a type, such as a <xref:PostSharp.Aspects.TypeLevelAspect>. On any other target, it causes the error <xref:la0251?text=LA0251>. It selects the methods declared by the target type that have a given name, including static and non-public methods, but not inherited methods. Each advice applies only to the methods whose signature matches its bound parameters (see <xref:advice-parameter-binding>). Several advices can therefore handle different overloads of the same method.

When <xref:PostSharp.Aspects.Advices.MatchPointcut.MatchParameterCount> is `true`, which is the default value, the pointcut also requires that the number of parameters of the method is the highest index of the `[Argument( index )]` parameters of the advice, plus one.

Only the `[Argument]` parameters that select the argument by its index are counted. When the advice has none, all the methods with the name are selected.

The following aspect validates the arguments of the overloads of the `Catalog.Add` method. The first advice applies to `Add( string )`, and the second one to `Add( string, string )`. The method `Add( int )` matches none of them, so it is left unchanged. The `nameof` operator names the method, so that the compiler checks the name.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutMatch.cs#example)]

When a program calls `Add( "" )`, `Add( "", "value" )` and `Add( 0 )`, it prints the following output:

[!code-text[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutMatch.txt)]

A target is skipped when it does not have the signature of the advice, for instance when an argument is missing or has an incompatible type, or when two bound values give conflicting type arguments. An error that does not come from the signature, such as a violated generic constraint, is still reported.

By default, an advice that applies to no method causes no error, for instance when the target type has no method of the name, or when no method of the name has the signature of the advice. Set <xref:PostSharp.Aspects.Advices.MatchPointcut.IsRequired> to `true` to make the build fail instead, with the error <xref:la0285?text=LA0285>, for each aspect instance whose advice applies to no method. Use it when the advice must not be lost silently, for instance after a method of the target type is renamed.

In the following example, the only `Add` method of `Inventory` has an `int` parameter, which does not match the `string` parameter of the advice:

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutMatchRequired.cs#example)]

The build fails with the following error:

[!code-text[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutMatchRequired.build.txt)]


## Self pointcut

The pointcut type <xref:PostSharp.Aspects.Advices.SelfPointcut> selects the target of the aspect. It is obsolete, and PostSharp ignores it: a group of advices without pointcut already applies to the target of the aspect, when its advices accept this kind of declaration. When they do not, for instance a method boundary advice in a <xref:PostSharp.Aspects.TypeLevelAspect>, PostSharp reports the error <xref:la0049?text=LA0049>, and the advice needs a pointcut.

Remove <xref:PostSharp.Aspects.Advices.SelfPointcut> from existing aspects. Before PostSharp 2027.0, it also gave an advice a group of its own. To do that now, set the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> property (see [Grouping advices](#grouping-advices)).


## Selecting targets programmatically

Pointcuts select the targets of the advices declared in the aspect class. PostSharp provides three ways to select targets with code instead of custom attributes:

* <xref:PostSharp.Aspects.Advices.MethodPointcut> selects the targets of a declared advice with a method of the aspect, as shown above. Use it when the advice is known in advance and only its targets depend on the code.
* <xref:PostSharp.Aspects.Advices.IAdviceProvider> provides, for each target of the aspect, advices that import or introduce members, such as an import of several fields of the target class into one aspect field, and requests for fields and dependencies. Use it when the advices themselves depend on the target. See <xref:advice-provider>.
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

PostSharp forms the groups of each class of the aspect separately, and it infers them in most aspects:

* An advice with a pointcut starts a group of its own. The group applies to the declarations that the pointcut selects.
* An advice without pointcut and without the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> property joins the group of its class that has no advice of its kind yet. When the class has no such group, the advice starts a group that applies to the target of the aspect. When the class has several, PostSharp reports the error <xref:la0230?text=LA0230>.
* An advice with the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> property joins the group of this name. The name is shared by the classes of the aspect, so an advice of a derived class can join a group of its base class.

The inference never looks at another class, and declaration order never chooses between two groups.

The following code groups an <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice> and an <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice>. `OnEntry` starts a group because it has a pointcut, and `OnExit` joins it, because the group has no exit advice yet. The `stopwatch` parameters of both advices are bound to the same variable: the entry advice assigns it, and the exit advice reads it.

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/CompositeAspects/AdviceGrouping.cs#advices)]

Some properties of the advice attributes apply to the whole group: the pointcut, <xref:PostSharp.Aspects.Advices.Advice.Description>, <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> and <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.UnsupportedTargetAction>. They can be set on any advice of the group, but on one advice only. When two advices of a group set one of them to different values, PostSharp reports the error <xref:la0231?text=LA0231>. When two advices of a group have a pointcut, it reports <xref:la0259?text=LA0259>.

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/AdviceProperties.cs#advice)]

An advice dependency (<xref:PostSharp.Aspects.Dependencies.AdviceDependencyAttribute>) can name any advice of a group to order the whole group.


### Several groups of the same family

When the class has several groups that could take an advice, set the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> property of the advices of each group to the same name. In the following example, the entry advices start two groups with their pointcuts, and the exit advice would find two groups without exit advice: <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> puts it in the group of the getters.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/PointcutSeveralGroups.cs#advices)]

The <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property, which named the master advice of the group, is obsolete. It keeps working, but set <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> on all the advices of the group instead.

Two groups of the same family that both apply to the target of the aspect are woven as two transformations of the same declaration, which is rarely what the author wants. PostSharp reports them with the warning <xref:la0257?text=LA0257>.

PostSharp reports the following errors and warnings about groups:

| Code | Severity | Meaning |
|------|----------|---------|
| <xref:la0049?text=LA0049> | Error | A group without pointcut cannot apply to the target of the aspect. Add a pointcut. |
| <xref:la0059?text=LA0059> | Error | The <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> or <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property puts two advices of the same kind in one group. |
| <xref:la0230?text=LA0230> | Error | Several groups of the class have no advice of the kind of an advice. Set the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> property. |
| <xref:la0231?text=LA0231> | Error | Two advices of one group set a property of the group to different values. |
| <xref:la0255?text=LA0255> | Warning | A <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> property selects the group that PostSharp infers anyway, and no derived class selects its name. |
| <xref:la0257?text=LA0257> | Warning | Two groups of one family apply to the target of the aspect. |
| <xref:la0258?text=LA0258> | Error | An advice sets both <xref:PostSharp.Aspects.Advices.GroupingAdvice.Group> and <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master>. |
| <xref:la0259?text=LA0259> | Error | Two advices of one group each have a pointcut. |


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.MulticastPointcut>
<br><xref:PostSharp.Aspects.Advices.MethodPointcut>
<br><xref:PostSharp.Aspects.Advices.MatchPointcut>
<br><xref:PostSharp.Aspects.Advices.SelfPointcut>
<br><xref:PostSharp.Aspects.Advices.GroupingAdvice.Group>

**Other Resources**

<xref:complex-aspects>
<br><xref:advices>
<br><xref:advice-parameter-binding>
<br><xref:advice-provider>
<br><xref:aspect-provider>
<br><xref:advice-ordering>
