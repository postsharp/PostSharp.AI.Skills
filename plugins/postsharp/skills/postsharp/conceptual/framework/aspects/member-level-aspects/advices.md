---
uid: advices
redirect_from:
  - custompatterns/aspects/compositeaspects/advices
  - framework/aspects/composite-aspects/advices
title: "Advice Methods"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how to write the advice methods of an aspect: the rules of their signature, the kinds of advices and the declarations they transform, the properties of the advice attributes, and the legacy AdviceArgs signature."
---
# Advice Methods

An *advice* is a method of the aspect class that adds a behavior to a declaration of the target code. A custom attribute on the method, such as <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice>, determines the kind of the advice, that is, when the method is invoked. You can choose any name for the method.

The same advice methods are used by member-level aspects (see <xref:member-level-aspects>) and by type-level and assembly-level aspects (see <xref:complex-aspects>). In a member-level aspect, the advices apply to the target of the aspect. In a type-level or assembly-level aspect, a pointcut selects the members to which an advice applies (see <xref:pointcuts>).


## Writing an advice method

An advice method follows these rules:

* It is public and returns `void`, except a <xref:PostSharp.Aspects.Advices.LocationValidationAdvice>, which returns an <xref:System.Exception>, and an <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice>, which returns a <xref:System.Threading.Tasks.Task>.
* It can be static or an instance method. In an <xref:PostSharp.Aspects.InstanceLevelAspect>, an advice that applies to static members must be static (error LA0107).
* It has the custom attribute of one advice kind, from the table below.
* Each of its parameters receives a value of the target, such as an argument, the return value or a variable shared with the other advices. A binding attribute on the parameter, such as `[Argument( 0 )]` or `[DeclarationName]`, selects the value. See <xref:advice-parameter-binding>.
* It can be generic. The type arguments are then inferred for each target from the bound values.

The following advice writes the name of the target method and the value of its first argument to the console. The name is a constant, so this advice allocates no memory, except to box the argument when it is of a value type.

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/AdviceMethod.cs#advice)]


## Advice kinds

The following table lists the kinds of advices that transform existing declarations. Most advices have a counterpart in a simple aspect. For instance, <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice> corresponds to <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)>. The documentation of the simple aspect describes the behavior of the advice in more detail. 

The advices of one row form a *family*. The advices of a family that apply to the same declarations are grouped and woven as one transformation (see [Grouping advices](xref:pointcuts#grouping-advices)). The last column says whether the advice accepts bound parameters.

| Advice type | Targets | Description | Bound parameters |
|-------------|---------|-------------|------------------|
| <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice><br><xref:PostSharp.Aspects.Advices.OnMethodSuccessAdvice><br><xref:PostSharp.Aspects.Advices.OnMethodExceptionAdvice><br><xref:PostSharp.Aspects.Advices.OnMethodExitAdvice><br><xref:PostSharp.Aspects.Advices.OnMethodYieldAdvice><br><xref:PostSharp.Aspects.Advices.OnMethodResumeAdvice> | Methods | These advices are equivalent to the advices of the aspect <xref:PostSharp.Aspects.OnMethodBoundaryAspect>. The target method is wrapped by a `try` / `catch` / `finally` construct. The yield and resume advices run around the `await` operators of async methods and the `yield return` statements of iterators.  | Yes |
| <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice> | Methods | This advice is equivalent to the aspect <xref:PostSharp.Aspects.MethodInterceptionAspect>. The body of the target method is replaced by a call to the advice, which can invoke the original implementation.  | Yes |
| <xref:PostSharp.Aspects.Advices.OnLocationGetValueAdvice><br><xref:PostSharp.Aspects.Advices.OnLocationSetValueAdvice> | Fields, properties | These advices are equivalent to the advices of the aspect <xref:PostSharp.Aspects.LocationInterceptionAspect>. Fields are changed into properties, and calls to the accessors are replaced by calls to the proper advice.  | Yes |
| <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice> | Types | This advice runs once an instance of the target type is constructed, whichever constructor ran. In a member-level aspect, the target type is the declaring type of the member. In an assembly-level aspect, the advice causes the error LA0088. | Yes |
| <xref:PostSharp.Aspects.Advices.LocationValidationAdvice> | Fields, properties, parameters | This advice is equivalent to the <xref:PostSharp.Aspects.ILocationValidationAspect`1.ValidateValue(`0,System.String,PostSharp.Reflection.LocationKind,PostSharp.Aspects.LocationValidationContext)> method of the <xref:PostSharp.Aspects.ILocationValidationAspect`1> aspect interface. It validates values assigned to their targets and returns an exception, which PostSharp throws, in case of error.  | No |
| <xref:PostSharp.Aspects.Advices.OnEventAddHandlerAdvice><br><xref:PostSharp.Aspects.Advices.OnEventRemoveHandlerAdvice><br><xref:PostSharp.Aspects.Advices.OnEventInvokeHandlerAdvice> | Events | These advices are equivalent to the advices of the aspect <xref:PostSharp.Aspects.EventInterceptionAspect>. Calls to `add` and `remove` semantics are replaced by calls to advices. When the event is fired, the `OnEventInvokeHandler` is invoked for each handler, instead of the handler itself.  | No |

Other advices introduce or import members instead of transforming existing ones. See <xref:code-injections> and <xref:members>.


## Advice properties

The advice attributes have properties that control how the advice is woven and how it is shown in Visual Studio:

* <xref:PostSharp.Aspects.Advices.Advice.Description> and <xref:PostSharp.Aspects.Advices.Advice.LinesOfCodeAvoided> describe the advice in CodeLens and tooltips (see <xref:customizing-aspect-description> and <xref:estimating-code-saving>).
* <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> and <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.UnsupportedTargetAction> control how the method boundary advices apply to async methods and iterators (see <xref:semantic-advising>). <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice> has the same two properties.

<xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> and <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.UnsupportedTargetAction> apply to the whole group, so PostSharp reads them from the master advice only. When PostSharp infers the group, the advice that sets one of these properties, or <xref:PostSharp.Aspects.Advices.Advice.Description>, becomes the master advice. When several advices set them, PostSharp reports the error LA0231. An advice that sets the <xref:PostSharp.Aspects.Advices.GroupingAdvice.Master> property cannot set them (error LA0135).

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/AdviceProperties.cs#advice)]


## Legacy AdviceArgs signature

Instead of bound parameters, an advice method can have the parameter of the corresponding method of the simple aspect: <xref:PostSharp.Aspects.MethodExecutionArgs> for the method boundary advices, <xref:PostSharp.Aspects.MethodInterceptionArgs> for <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, <xref:PostSharp.Aspects.LocationInterceptionArgs> for the location interception advices, and so on. This object is allocated on the heap on each call, and it gives access to the values of the target as <xref:System.Object>.

This signature is an obsolete practice, supported for compatibility with existing aspects. Prefer bound parameters. To convert an existing aspect, see <xref:modernizing-aspects>.

The event interception advices do not accept bound parameters, so they must use this signature. <xref:PostSharp.Aspects.Advices.LocationValidationAdvice> has its own signature, the one of the <xref:PostSharp.Aspects.ILocationValidationAspect`1.ValidateValue(`0,System.String,PostSharp.Reflection.LocationKind,PostSharp.Aspects.LocationValidationContext)> method. In the method boundary, method interception and location interception advices, the <xref:PostSharp.Aspects.AdviceArgs> parameter can also be the first parameter, followed by bound parameters.

The following advice of an <xref:PostSharp.Aspects.EventLevelAspect> intercepts the `add` accessor of the target event:

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/AdviceEventAddHandler.cs#advice)]


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices>
<br><xref:PostSharp.Aspects.Advices.Advice>
<br><xref:PostSharp.Aspects.Advices.AdviceParameterAttribute>

**Other Resources**

<xref:member-level-aspects>
<br><xref:advice-parameter-binding>
<br><xref:pointcuts>
<br><xref:modernizing-aspects>
<br><xref:advice-ordering>
<br><xref:simple-aspects>
