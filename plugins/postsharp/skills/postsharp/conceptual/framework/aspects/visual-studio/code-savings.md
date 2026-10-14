---
uid: estimating-code-saving
redirect_from:
  - custompatterns/aspects/intellisense/estimating-code-saving
title: "Estimating Code Savings"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp, a tool for Aspect-Oriented Programming, estimates lines of handwritten code saved by using aspects during build time. Users can adjust these estimates for more accuracy."
---
# Estimating Code Savings

During the build, PostSharp attempts to estimate how many lines of handwritten code were avoided thanks to aspects. By default, PostSharp considers that 2 lines of code are saved every time an advice is applied to a target. This is of course a very rough estimate. You can add information to your aspects and advices to make the estimate more accurate.

> [!TIP]
> When you add a code saving estimate, ask yourself the following question: how much code would an intelligent developer have written if they had to implement the same feature without PostSharp, using the best possible strategy? Do not assume that the strategy you took to implement the feature with an aspect would be the same as the strategy for handwritten code.


## Simple aspects

*Simple aspects* are aspects built by deriving from a base class such as <xref:PostSharp.Aspects.OnMethodBoundaryAspect> and overriding virtual methods of the base class, such as <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)>. They are described in the section <xref:simple-aspects>. 

By default, PostSharp estimates that 2 lines of handwritten code are avoided for each advice method that you override, every time the aspect is applied to a target.

To override the default value, add the <xref:PostSharp.Aspects.LinesOfCodeAvoidedAttribute> custom attribute to the aspect class. The argument of the custom attribute constructor must be set to the number of lines of handwritten code avoided every time the aspect is applied to a target. 

The following code snippet shows how to specify that 4 lines of code are avoided every time the aspect is applied. If the aspect is applied to 100 methods, PostSharp will estimate that 400 lines of handwritten code have been avoided.

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Intellisense/LinesOfCodeAvoided.cs#example)]


## Aspects with advice methods

Member-level, type-level and assembly-level aspects do not override methods of a base class. They declare advice methods with custom attributes such as <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice>, and possibly pointcuts such as <xref:PostSharp.Aspects.Advices.MethodPointcut> (see <xref:member-level-aspects> and <xref:complex-aspects>). Unlike simple aspects, these aspects can have several advices. 

By default, PostSharp estimates that 2 lines of handwritten code are avoided for each advice, every time the advice is applied to a target. Some advices may have different default values. For instance, the <xref:PostSharp.Aspects.Advices.IntroduceInterfaceAttribute> advice counts 2 lines of code per introduced interface method, and the <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute> advice counts 1 line of code per introduced member. 

You can still use the aspect-level <xref:PostSharp.Aspects.LinesOfCodeAvoidedAttribute> custom attribute. It will increment the estimated number of avoided lines of code every time the aspect is applied to a target. However, to provide more relevant estimates, you need to provide code saving information at advice level. 

To specify how many lines of handwritten code are avoided every time an advice is applied to a target, specify the <xref:PostSharp.Aspects.Advices.Advice.LinesOfCodeAvoided> property of the advice custom attribute. 

The following code snippet shows how to specify that 1 line of code is avoided every time the advice is applied. Suppose that the aspect is applied to 100 classes and each class has on average 5 non-abstract instance properties. In this situation, PostSharp will estimate that 500 lines of handwritten code have been avoided.

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/Intellisense/AdviceLinesOfCodeAvoided.cs#example)]


## Adding code saving hints programmatically

In the previous sections, we described how to add code saving hints declaratively using custom attributes. Sometimes declarative estimations are not accurate enough. To learn how to add programmatic hints, see <xref:programmatic-tooltip>. 

## See Also

**Reference**

<xref:PostSharp.Aspects.LinesOfCodeAvoidedAttribute>
<br><xref:PostSharp.Aspects.Advices.Advice.LinesOfCodeAvoided>
