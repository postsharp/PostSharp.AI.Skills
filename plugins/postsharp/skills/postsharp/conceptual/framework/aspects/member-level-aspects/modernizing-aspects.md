---
uid: modernizing-aspects
title: "Modernizing Your Custom Aspects"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains why and how to convert an aspect that receives MethodExecutionArgs, MethodInterceptionArgs or LocationInterceptionArgs into an aspect whose advices receive typed, bound parameters, with a mapping from the members of these classes to binding attributes."
---
# Modernizing Your Custom Aspects

Aspects derived from <xref:PostSharp.Aspects.OnMethodBoundaryAspect>, <xref:PostSharp.Aspects.MethodInterceptionAspect> or <xref:PostSharp.Aspects.LocationInterceptionAspect> receive their context through an <xref:PostSharp.Aspects.AdviceArgs> object, such as <xref:PostSharp.Aspects.MethodExecutionArgs>. This object is allocated on the heap on each call, so it adds pressure on the garbage collector. It is also weakly typed: arguments and return values are exposed as <xref:System.Object>, so the aspect must cast them, and values of value types are boxed. With advice parameter binding, an advice instead declares the values it needs as typed parameters (see <xref:advice-parameter-binding>). This article explains why and how to convert an existing aspect to bound parameters.


## Why modernize

Bound parameters have the following benefits:

* **Performance**. The woven code does not allocate an <xref:PostSharp.Aspects.AdviceArgs> object on each call, and it does not allocate an <xref:PostSharp.Aspects.Arguments> object unless the advice asks for all the arguments. Arguments and return values of value types are not boxed when the parameter has their type or is generic.
* **Type safety**. The advice receives typed values instead of <xref:System.Object> values that it must cast.
* **Build-time validation**. PostSharp checks the signature of the advice against each target. An argument that is missing or that has an incompatible type causes a build error, not an <xref:System.InvalidCastException> or an <xref:System.ArgumentOutOfRangeException> at run time. With <xref:PostSharp.Aspects.Advices.MatchPointcut>, the advice can instead skip the targets that do not match.
* **Readable code**. The signature of the advice shows which values the advice uses.

Keep the <xref:PostSharp.Aspects.AdviceArgs> signature in the following cases:

* The aspect intercepts events, validates locations or uses <xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvokeAsync(PostSharp.Aspects.MethodInterceptionArgs)>. These advices do not accept bound parameters.
* The aspect reads the value yielded by an iterator, which only <xref:PostSharp.Aspects.MethodExecutionArgs.YieldValue> gives.
* The aspect is not on a performance-sensitive path, and you prefer the simpler programming model of a simple aspect.

For the complete list, see the [limitations](xref:advice-parameter-binding#limitations).


## How to modernize an aspect

To convert a simple aspect to an aspect with bound parameters:

1. Change the base class of the aspect from the simple aspect class to the corresponding level aspect class:
   * <xref:PostSharp.Aspects.OnMethodBoundaryAspect> and <xref:PostSharp.Aspects.MethodInterceptionAspect> become <xref:PostSharp.Aspects.MethodLevelAspect>.
   * <xref:PostSharp.Aspects.LocationInterceptionAspect> becomes <xref:PostSharp.Aspects.LocationLevelAspect>.

   The multicasting properties and the `CompileTimeValidate`, `CompileTimeInitialize` and `RuntimeInitialize` methods are defined on these classes too, so they do not change.

2. Change each overridden method into an advice method. Remove the `override` keyword, and add the advice attribute that corresponds to the method, for instance <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice> for `OnEntry`. See the table of [advice kinds](xref:advices#advice-kinds).

3. Do not add pointcuts. Advices without pointcut apply to the target of the aspect, and the advices of one family form one group, which is woven as a single `try` / `catch` / `finally` construct, like the simple aspect (see <xref:member-level-aspects>).

4. Move the properties of the simple aspect that control the weaving to one of the advices, which becomes the master advice of the group. For a method boundary aspect, these properties are <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> and <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.UnsupportedTargetAction>. A method interception aspect has the same two properties, which move to the <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>. Set these properties on one advice only: when several advices set them, PostSharp cannot infer the group and reports the error LA0231.

5. Replace the <xref:PostSharp.Aspects.AdviceArgs> parameter of each advice with bound parameters, following the table below.

6. Build the project and fix the errors that PostSharp reports on the advices. For the list of messages, see [Build messages](xref:advice-parameter-binding#build-messages).


## Mapping AdviceArgs members to bound parameters

### MethodExecutionArgs

| Member of <xref:PostSharp.Aspects.MethodExecutionArgs> | Bound parameter |
|---------------------------------|-----------------|
| <xref:PostSharp.Aspects.MethodExecutionArgs.Arguments>`[i]` | `[Argument( i )] T value`, or `ref` to change the argument. The argument can also be selected by its name or its type. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Arguments> | `[Arguments] Arguments arguments`. This allocates an object on each call. |
| <xref:PostSharp.Aspects.AdviceArgs.Instance> | `[This] object instance`, or the type of the target class. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Method> | `[Declaration] MethodBase method` |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Method>`.Name` | `[DeclarationName] string name` |
| <xref:PostSharp.Aspects.AdviceArgs.DeclarationIdentifier> | `[DeclarationIdentifier] DeclarationIdentifier identifier` |
| <xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag> | `[State( StateScope.MethodInvocation )] T state`: `out` in the entry advice, by value in the other advices. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue> | `[ReturnValue] T value`: by value to read it, `ref` to replace it in the success and exit advices, `out` to set it in the entry and exception advices. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> | `[Exception] Exception exception` in the exception advice, or `ref` to replace it. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> | `[FlowBehavior] out FlowBehavior flowBehavior` in the entry, exception and resume advices. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.YieldValue> | No equivalent. |

For async methods, the method boundary advices also accept `[AsyncCallId]`, `[CurrentTask]`, `[AwaitedTask]`, `[AwaitedMethod]` and `[Awaiter]`. These values are not available through <xref:PostSharp.Aspects.MethodExecutionArgs>.


### MethodInterceptionArgs

| Member of <xref:PostSharp.Aspects.MethodInterceptionArgs> | Bound parameter |
|-----------------------------------|-----------------|
| <xref:PostSharp.Aspects.MethodInterceptionArgs.Proceed> | `returnValue = (T) binding.Invoke( ref instance, arguments )`, with the parameters `[This] object instance`, `[Binding] IMethodBinding binding`, `[Arguments] Arguments arguments` and `[ReturnValue] out T returnValue`. |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.Invoke(PostSharp.Aspects.Arguments)> | `binding.Invoke( ref instance, otherArguments )` |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.Arguments> | `[Arguments] Arguments arguments`, or `[Argument( i )] T value` to read one argument. |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.ReturnValue> | `[ReturnValue] out T returnValue` |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.Method> | `[Declaration] MethodBase method` |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.Binding> | `[Binding] IMethodBinding binding` |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.ProceedAsync> | No equivalent. <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice> does not accept bound parameters. |


### LocationInterceptionArgs

In the following table, `binding` is a `[Binding] ILocationBinding<T>` parameter, and `instance` is a `[This] object` parameter. On an instance field or property of a struct, the `instance` parameter receives a boxed copy, so the value is stored in the copy and lost. Apply such an aspect only to classes and static members, or keep the <xref:PostSharp.Aspects.LocationInterceptionArgs> signature for structs.

| Member of <xref:PostSharp.Aspects.LocationInterceptionArgs> | Bound parameter |
|-------------------------------------|-----------------|
| <xref:PostSharp.Aspects.Internals.LocationLevelAdviceArgs.Value> | `[LocationValue] out T value` in a get advice, `[LocationValue] T value` in a set advice. |
| <xref:PostSharp.Aspects.LocationInterceptionArgs.ProceedGetValue> | `value = binding.GetValue( instance )` |
| <xref:PostSharp.Aspects.LocationInterceptionArgs.ProceedSetValue> | `binding.SetValue( instance, value )` |
| <xref:PostSharp.Aspects.LocationInterceptionArgs.GetCurrentValue> | `binding.GetValue( instance )` |
| <xref:PostSharp.Aspects.LocationInterceptionArgs.SetNewValue(System.Object)> | `binding.SetValue( instance, newValue )` |
| <xref:PostSharp.Aspects.Internals.LocationLevelAdviceArgs.Location> | `[Declaration] LocationInfo location` |
| <xref:PostSharp.Aspects.Internals.LocationLevelAdviceArgs.LocationName> | `[DeclarationName] string name` |
| <xref:PostSharp.Aspects.LocationInterceptionArgs.Binding> | `[Binding] ILocationBinding<T> binding` |
| <xref:PostSharp.Aspects.LocationInterceptionArgs.Index> | No equivalent. |


## Example of a method boundary aspect

The following aspect logs the execution of a method with an <xref:PostSharp.Aspects.OnMethodBoundaryAspect>. It reads the first argument and the return value as objects, and it stores a <xref:System.Diagnostics.Stopwatch> in <xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag>.

```csharp
[PSerializable]
public sealed class LogAttribute : OnMethodBoundaryAspect
{
    public override void OnEntry( MethodExecutionArgs args )
    {
        Console.WriteLine( "Entering {0}({1}).", args.Method.Name, args.Arguments[0] );
        args.MethodExecutionTag = Stopwatch.StartNew();
    }

    public override void OnSuccess( MethodExecutionArgs args )
    {
        Console.WriteLine( "{0} returned {1}.", args.Method.Name, args.ReturnValue );
    }

    public override void OnException( MethodExecutionArgs args )
    {
        Console.WriteLine( "{0} failed: {1}", args.Method.Name, args.Exception.Message );
    }

    public override void OnExit( MethodExecutionArgs args )
    {
        var stopwatch = (Stopwatch) args.MethodExecutionTag;
        Console.WriteLine( "{0} took {1} ms.", args.Method.Name, stopwatch.ElapsedMilliseconds );
    }
}
```

The following code is the same aspect with bound parameters. The name of the method is a constant, the stopwatch is a typed variable shared by the advices, and the success advice is generic, so the return value is passed without boxing. The first argument is still boxed because the parameter is of type <xref:System.Object>. To avoid this, make the advice generic and bind the argument to a parameter of type `T`.

```csharp
[PSerializable]
public sealed class LogAttribute : MethodLevelAspect
{
    [OnMethodEntryAdvice]
    public void OnEntry( [DeclarationName] string methodName,
                         [Argument( 0 )] object argument,
                         [State( StateScope.MethodInvocation )] out Stopwatch stopwatch )
    {
        Console.WriteLine( "Entering {0}({1}).", methodName, argument );
        stopwatch = Stopwatch.StartNew();
    }

    [OnMethodSuccessAdvice]
    public void OnSuccess<T>( [DeclarationName] string methodName, [ReturnValue] T returnValue )
    {
        Console.WriteLine( "{0} returned {1}.", methodName, returnValue );
    }

    [OnMethodExceptionAdvice]
    public void OnException( [DeclarationName] string methodName, [Exception] Exception exception )
    {
        Console.WriteLine( "{0} failed: {1}", methodName, exception.Message );
    }

    [OnMethodExitAdvice]
    public void OnExit( [DeclarationName] string methodName,
                        [State( StateScope.MethodInvocation )] Stopwatch stopwatch )
    {
        Console.WriteLine( "{0} took {1} ms.", methodName, stopwatch.ElapsedMilliseconds );
    }
}
```

The original aspect could be applied to any method. The modernized aspect requires a method with at least one parameter: on a method without parameters, `[Argument( 0 )]` causes the error LA0225. On a method that returns `void`, the `returnValue` parameter receives the default value.


## Example of a location interception aspect

The following aspect converts the value assigned to a string property of a class to uppercase, first with a <xref:PostSharp.Aspects.LocationInterceptionAspect>:

```csharp
[PSerializable]
public sealed class UpperCaseAttribute : LocationInterceptionAspect
{
    public override void OnSetValue( LocationInterceptionArgs args )
    {
        args.Value = ((string) args.Value)?.ToUpperInvariant();
        args.ProceedSetValue();
    }
}
```

And then with bound parameters:

```csharp
[PSerializable]
public sealed class UpperCaseAttribute : LocationLevelAspect
{
    [OnLocationSetValueAdvice]
    public void OnSetValue( [This] object instance,
                            [LocationValue] string value,
                            [Binding] ILocationBinding<string> binding )
    {
        binding.SetValue( instance, value?.ToUpperInvariant() );
    }
}
```


## Differences in behavior

Check the following points after the conversion:

* An advice is checked against the signature of each target. A target that the original aspect accepted can cause an error, for instance when an argument or a return value has an incompatible type. Restrict the targets with multicasting or `CompileTimeValidate`, or use <xref:PostSharp.Aspects.Advices.MatchPointcut> to skip them.
* `[CurrentTask]` is `null` until the async method yields.
* When an advice keeps its <xref:PostSharp.Aspects.MethodExecutionArgs> parameter and adds a `[ReturnValue]`, `[FlowBehavior]` or `[Exception]` parameter that writes the same value, the bound parameter takes precedence, and PostSharp reports a warning (LA0234, LA0243 or LA0248).
* In a method interception advice, a value written to an `out` or `ref` `[Argument]` parameter does not reach the intercepted method (warning LA0235). Change the <xref:PostSharp.Aspects.Arguments> object passed to the binding instead.
* Advices of a simple aspect are always grouped. In a member-level aspect, the advices of one family are grouped too. Advices of different families, such as a method boundary advice and a method interception advice, are separate transformations, and their order determines the generated code (see <xref:advice-ordering>).


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.AdviceParameterAttribute>
<br><xref:PostSharp.Aspects.MethodLevelAspect>
<br><xref:PostSharp.Aspects.LocationLevelAspect>

**Other Resources**

<xref:advice-parameter-binding>
<br><xref:advices>
<br><xref:simple-aspects>
