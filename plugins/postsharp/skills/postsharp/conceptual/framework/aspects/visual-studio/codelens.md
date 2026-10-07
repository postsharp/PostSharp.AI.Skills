---
uid: customizing-aspect-description
redirect_from:
  - custompatterns/aspects/intellisense/customizing-aspect-description
title: "Customizing Aspect Description in CodeLens and Tooltips"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides guidance on customizing aspect descriptions in CodeLens and tooltips using Visual Studio Tools for Metalama and PostSharp, including instructions for both simple aspects and aspects with advice methods."
---
# Customizing Aspect Description in CodeLens and Tooltips
When you position the mouse cursor over a declaration that has been enhanced by an aspect, Visual Studio Tools for Metalama and PostSharp add a description of the aspect to CodeLens, and optionally to the IntelliSense tooltip. The description that PostSharp generates by default is sometimes not very helpful. To make the CodeLens and IntelliSense description of your aspect more understandable for its users, you should override the default description.


## Simple aspects

*Simple aspects* are aspects built by deriving from a base class such as <xref:PostSharp.Aspects.OnMethodBoundaryAspect> and overriding virtual methods of the base class, such as <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)>. They are described in the section <xref:simple-aspects>. 

To set the description of a simple aspect, add the <xref:PostSharp.Aspects.AspectDescriptionAttribute> custom attribute to the aspect class. 

This is illustrated in the following code snippet.

```csharp
[PSerializable]
    [AspectDescription("Applies the exception handling policy")]
    public sealed class ExceptionHandlerAttribute : OnExceptionAspect
    {
        public override void OnException( MethodExecutionArgs eventArgs )
        {
            if ( !ExceptionHandler.OnException( eventArgs.Exception ) )
            {
                eventArgs.FlowBehavior = FlowBehavior.Continue;
            }
        }
    }
```


## Aspects with advice methods

Member-level, type-level and assembly-level aspects do not override methods of a base class. They declare advice methods with custom attributes such as <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice>, and possibly pointcuts such as <xref:PostSharp.Aspects.Advices.MethodPointcut> (see <xref:member-level-aspects> and <xref:complex-aspects>). Unlike simple aspects, these aspects can have several advices. 

With these aspects, you should add a description to every group of advices. You can do that by setting the <xref:PostSharp.Aspects.Advices.Advice.Description> property of the advice custom attribute. When PostSharp infers the group, set the description on one advice of the group only: several descriptions in one inferred group cause the error LA0231 (see <xref:pointcuts>). 

The following code snippet illustrates how to set the description of the advice. This description will appear in CodeLens and the IntelliSense tooltip of each property affected by this advice.

```csharp
[OnLocationSetValueAdvice( Description="Persists the property to disk." ),
 MulticastPointcut( Targets = MulticastTargets.Property,
                    Attributes = MulticastAttributes.Instance | MulticastAttributes.NonAbstract )]
public void OnPropertySet<T>( [This] object instance, [LocationValue] T value, [Binding] ILocationBinding<T> binding )
{
    // Details skipped.
}
```


