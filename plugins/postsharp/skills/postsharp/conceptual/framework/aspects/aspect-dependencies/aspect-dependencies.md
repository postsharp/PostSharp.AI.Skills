---
uid: aspect-dependencies
redirect_from:
  - custompatterns/aspects/dependencies/aspect-dependencies
title: "Coping with Several Aspects on the Same Target"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how PostSharp provides a framework to manage dependencies between aspects in aspect-oriented programming, preventing conflicts and ensuring predictable order of execution. It also covers aspect dependency custom attributes and types of aspect dependency relationships."
---
# Coping with Several Aspects on the Same Target

As the team learns aspect-oriented programming and starts adding more aspects to projects, the chance increases that several aspects are added to the same element of code. This could be a major source of trouble if PostSharp did not provide a robust framework to detect and prevent conflicts between aspects:

* Most aspects need to be ordered. For instance, an authorization aspect must be executed before a caching aspect. 

* Even if some aspects do not require a specific order, it is good to have them applied in a predictable order. Otherwise, some code that works today may break tomorrow just because aspects were applied in a different order. 

* Some aspects conflict: they cannot be on the same target, or not in a given order. For instance, it does not make sense to persist an object using two different aspects: one would persist to the database, the other to the registry. 

* Some aspects require other aspects to be applied. For instance, an aspect changing the mouse pointer to an hourglass requires the method to execute asynchronously, otherwise the pointer shape will never be updated. 

PostSharp addresses these issues by making it possible to add dependencies between aspects. The aspect dependency framework is implemented in the namespace <xref:PostSharp.Aspects.Dependencies>. 

> [!NOTE]
> The aspect dependency framework is not related to the notion of dependency injection.


## Aspect dependency custom attributes

You can express dependencies of an aspect by annotating the aspect class with custom attributes derived from the type <xref:PostSharp.Aspects.Dependencies.AspectDependencyAttribute>. Several derived types are available; every type matches other aspects according to different criteria. 


#### Types of aspect dependency custom attributes

| Attribute type | Description |
|----------------|-------------|
| <xref:PostSharp.Aspects.Dependencies.AspectTypeDependencyAttribute> | This custom attribute expresses a dependency with a well-known aspect class. |
| <xref:PostSharp.Aspects.Dependencies.AspectRoleDependencyAttribute> | This custom attribute expresses a dependency with any aspect classes enrolled in a given role. Its dual is <xref:PostSharp.Aspects.Dependencies.ProvideAspectRoleAttribute>: this custom attribute enrolls an aspect class into a role. A role is simply a string. Whenever possible, consider using one of the roles defined in the class <xref:PostSharp.Aspects.Dependencies.StandardRoles>.  |
| <xref:PostSharp.Aspects.Dependencies.AspectEffectDependencyAttribute> | This custom attribute expresses a dependency with any aspect that has a specific effect on the source code or the control flow. Effects are represented as a string, whose valid values are listed in the type <xref:PostSharp.Aspects.Dependencies.StandardEffects>. Effects are provisioned by the aspect weaver on the basis of a rough analysis of what the aspect may do; aspect developers cannot assign new effects to aspects. However, they can waive effects by using the custom attribute <xref:PostSharp.Aspects.Dependencies.WaiveAspectEffectAttribute>. For instance, an aspect developer can specify that a trace attribute has no effect at all; this aspect will commute with any other aspect (see below).  |

All these custom attributes have similar structure and members. The first parameter of their constructor, of type <xref:PostSharp.Aspects.Dependencies.AspectDependencyAction>, determines the kind of dependency relationship added between the current aspect and the aspects matched by the custom attribute. 

PostSharp supports the following kinds of relationships:


#### Kinds of aspect dependency relationships

| Action | Description |
|--------|-------------|
| `Order` | The dependency expresses an order relationship. The second constructor parameter of the custom attribute, of type <xref:PostSharp.Aspects.Dependencies.AspectDependencyPosition> (with values `Before` or `After`), must be specified. The custom attribute determines the position of the current aspect with respect to matched aspects.  |
| `Require` | The dependency expresses a requirement. PostSharp will issue a compile-time error if the requirement is not satisfied for any target of the current aspect. The second constructor parameter of the custom attribute, of type <xref:PostSharp.Aspects.Dependencies.AspectDependencyPosition>, is optional. If specified, the current aspect must be before or after the matching aspect.  |
| `Conflict` | The dependency expresses a conflict. PostSharp will issue a compile-time error if any aspect matching the dependency rule is present on any target of the current aspect. The second constructor parameter of the custom attribute, of type <xref:PostSharp.Aspects.Dependencies.AspectDependencyPosition>, is optional. If specified, an error is issued only if the current aspect is before or after a matching aspect.  |
| `Commute` | The dependency specifies that the current aspect is commutable with any matching aspect. When aspects are commutable, PostSharp does not issue any warning if they are not strongly ordered. |

Custom attribute types and values of the enumeration <xref:PostSharp.Aspects.Dependencies.AspectDependencyAction> are orthogonal; they can be freely combined. 


## Examples


### Using role-based dependencies

The following code shows how three aspects can be ordered without having explicit knowledge of each other. Each aspect provides a different role, and defines dependencies with respect to other roles.

```csharp
[ProvideAspectRole( StandardRoles.Threading )]
[AspectRoleDependency(AspectDependencyAction.Order, AspectDependencyPosition.Before, "UI")]
public sealed class BackgroundAttribute : MethodInterceptionAspect
{
  // Details skipped
}

[ProvideAspectRole( StandardRoles.ExceptionHandling )]
[AspectRoleDependency( AspectDependencyAction.Order, AspectDependencyPosition.After, StandardRoles.Threading )]
[AspectRoleDependency(AspectDependencyAction.Order, AspectDependencyPosition.After, "UI")]
public sealed class ExceptionDialogAttribute : OnExceptionAspect
{
  // Details skipped
}

[ProvideAspectRole("UI")]
public sealed class StatusTextAttribute : OnMethodBoundaryAspect
{
  // Details skipped
}
```


### Using effect-based dependencies

The following code shows how to prevent an authorization aspect from being executed after an aspect that may change the control flow and skip the execution of the method, such as a caching aspect. Then, it shows how the aspect `BackgroundAttribute` can opt out of this effect, because the aspect developer knows that this aspect does not skip the execution of the method, but only defers it. 

```csharp
[AspectEffectDependency( AspectDependencyAction.Conflict, AspectDependencyPosition.After, 
                         StandardEffects.ChangeControlFlow )]
public sealed class AuthorizationAttribute : OnMethodBoundaryAspect
{
  // Details skipped.
}

[WaiveAspectEffect(StandardEffects.ChangeControlFlow)]
public sealed class BackgroundAttribute : MethodInterceptionAspect
{
  // Details skipped
}
```


## Deferring ordering to aspect users

By adding dependencies to the aspect class, the aspect developer specifies the order of execution of aspects in a fully static way. The same order is used for every element of code to which aspects apply. Most of the time, this behavior is desirable, but there may be situations where we want to defer ordering to users of our aspects.

Aspect users can influence the order of execution of an aspect by setting the aspect property <xref:PostSharp.Aspects.Aspect.AspectPriority>, typically when using the aspect custom attribute (the same property is available in the configuration object as <xref:PostSharp.Aspects.Configuration.AspectConfiguration.AspectPriority>, see <xref:aspect-configuration>). 

Setting the <xref:PostSharp.Aspects.Aspect.AspectPriority> property on an aspect adds an ordering dependency between this aspect and all other aspects where the same property has been set. Therefore, aspect priorities complement, and do not replace, other ordering dependencies. The aspect developer may specify only the vital aspect dependencies (that is, under-specify aspect ordering), and let the aspect user complete the ordering with priorities. 

> [!CAUTION]
> Do not confuse the property <xref:PostSharp.Aspects.Aspect.AspectPriority> with <xref:PostSharp.Extensibility.MulticastAttribute.AttributePriority>. The latter determines the order in which several custom attributes of the same type are processed by the <xref:PostSharp.Extensibility.MulticastAttribute> engine. The former determines the order in which the aspects are executed at run time. 


## Adding dependencies to third-party aspects

If you are using aspects provided by several third-party vendors who do not know about each other, you may need to solve conflicts on your own.

You can do that by adding any custom attribute derived from <xref:PostSharp.Aspects.Dependencies.AspectDependencyAttribute> at assembly level, and using the property <xref:PostSharp.Aspects.Dependencies.AspectDependencyAttribute.TargetType> to specify to which aspect class the dependency applies. 

Here is an example:

```csharp
[assembly: AspectTypeDependency( AspectDependencyAction.Order, AspectDependencyPosition.Before,
                                 typeof(Vendor1.TraceAspect), TargetType = typeof(Vendor2.ExceptionHandlingAspect) )]
```

## See Also

**Reference**

<xref:PostSharp.Aspects.Dependencies>
