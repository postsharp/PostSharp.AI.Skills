---
uid: advice-parameter-binding
title: "Advice Parameters"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how an advice method receives the values it needs, such as arguments, the return value, the exception or a variable shared between advices, as typed parameters annotated with binding attributes, and lists the bindings that each kind of advice accepts."
---
# Advice Parameters

An advice method receives the values that it needs from the target as parameters. Each parameter has a custom attribute, called a *binding attribute*, that selects the value: an argument, the instance, the return value, the exception, or a variable shared by several advices. This mechanism is called *advice parameter binding*.

Bound parameters have the following benefits over a single <xref:PostSharp.Aspects.AdviceArgs> parameter, such as <xref:PostSharp.Aspects.MethodExecutionArgs>:

* The woven code computes only the values that the advice declares. No <xref:PostSharp.Aspects.AdviceArgs> object is allocated on each call.
* The parameters are typed. An argument of a value type is not boxed when the parameter has the type of the argument or is generic.
* PostSharp checks the signature of the advice when it builds the aspect, and again for each target. An advice that does not fit a target causes a build error instead of an exception at run time.

The binding attributes derive from <xref:PostSharp.Aspects.Advices.AdviceParameterAttribute> and are in the <xref:PostSharp.Aspects.Advices> namespace. For an introduction to advices and pointcuts, see <xref:advices>. To convert an existing aspect, see <xref:modernizing-aspects>.


## Example

The following aspect measures the duration of a method and writes it to the console with the value of the first argument. The entry advice starts a <xref:System.Diagnostics.Stopwatch> and stores it in a variable bound by <xref:PostSharp.Aspects.Advices.StateAttribute>. The advices have no pointcut, so they apply to the target method of the aspect and form one group (see [How it works](xref:member-level-aspects#how-it-works)). The `stopwatch` parameter of the exit advice is therefore bound to the same variable.

```csharp
using System;
using System.Diagnostics;
using PostSharp.Aspects;
using PostSharp.Aspects.Advices;
using PostSharp.Serialization;

[PSerializable]
public sealed class StopwatchAttribute : MethodLevelAspect
{
    [OnMethodEntryAdvice]
    public void OnEntry( [State( StateScope.MethodInvocation )] out Stopwatch stopwatch )
    {
        stopwatch = Stopwatch.StartNew();
    }

    [OnMethodExitAdvice]
    public void OnExit( [DeclarationName] string methodName,
                        [Argument( 0 )] object argument,
                        [State( StateScope.MethodInvocation )] Stopwatch stopwatch )
    {
        Console.WriteLine( "{0}({1}) took {2} ms.", methodName, argument, stopwatch.ElapsedMilliseconds );
    }
}
```


## Rules of an advice signature

An advice method with bound parameters follows these rules:

* The method is public. It can be static or an instance method, and it returns `void`.
* Each parameter has exactly one binding attribute.
* Only the first parameter can have no binding attribute. It must then be of the <xref:PostSharp.Aspects.AdviceArgs> type of the advice, for instance <xref:PostSharp.Aspects.MethodExecutionArgs> in a method boundary advice. The other parameters are bound as usual. An <xref:PostSharp.Aspects.AdviceArgs> parameter is an obsolete practice, supported for compatibility with existing aspects. Prefer bound parameters (see <xref:modernizing-aspects>). <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice> accepts no <xref:PostSharp.Aspects.AdviceArgs> parameter.
* A parameter can be passed by value, or as `in`, `ref` or `out`. Each binding attribute accepts some of these modes. A `ref` or `out` parameter gives write access to the bound value, for instance to replace an argument or the return value.
* The type of a parameter passed by value can be the type of the bound value, or a base type of it when the value is of a reference type. With `[Argument]`, `[ReturnValue]` and `[This]`, a parameter of a reference type, such as <xref:System.Object>, receives a boxed copy of a value of a value type. The other bindings require the types listed in the table below. A parameter passed as `in`, `ref` or `out` must have exactly the type of the bound value.
* The method can be generic. The type arguments are then inferred for each target. See [Generic advice methods](#generic-advice-methods).

Only the binding attributes of PostSharp are supported. A custom attribute that you derive from <xref:PostSharp.Aspects.Advices.AdviceParameterAttribute> causes an error.


## Binding attributes

The following table lists the binding attributes, the value they bind and the parameter modes they accept. The documentation of each attribute gives the details.

| Attribute | Bound value | Parameter type | Modes |
|-----------|-------------|----------------|-------|
| <xref:PostSharp.Aspects.Advices.ArgumentAttribute> | An argument of the target method, selected by its index, its name or its type. | The type of the argument, or a base type. | Any. With `ref` or `out`, the method body and the next advices see the changes. |
| <xref:PostSharp.Aspects.Advices.ArgumentsAttribute> | All the arguments, in a new <xref:PostSharp.Aspects.Arguments> object allocated on each call. | <xref:PostSharp.Aspects.Arguments> or <xref:System.Object>. | By value. |
| <xref:PostSharp.Aspects.Advices.ThisAttribute> | The instance of the target (`this`). `null` or the default value on a static target, unless <xref:PostSharp.Aspects.Advices.ThisAttribute.IsRequired> is `true`. | The type of the instance, or a base type. | By value. `ref` or `in` only on a struct. |
| <xref:PostSharp.Aspects.Advices.StateAttribute> | A variable shared by the advices of a group for one execution of the target method. Its initial value is the default value of its type. | Any type, the same in all the advices that bind the slot. | Any. |
| <xref:PostSharp.Aspects.Advices.ReturnValueAttribute> | The return value of the target method. In a method boundary advice applied semantically to an async method, the result of the task. | The return type. | Depends on the advice. See [Reading and setting the return value](#reading-and-setting-the-return-value). |
| <xref:PostSharp.Aspects.Advices.ExceptionAttribute> | The exception thrown by the target method. | <xref:System.Exception>, or <xref:System.Object> by value. | Any. With `ref` or `out`, the advice can replace the exception. |
| <xref:PostSharp.Aspects.Advices.FlowBehaviorAttribute> | A <xref:PostSharp.Aspects.FlowBehavior> variable that the advice sets to change the control flow. | <xref:PostSharp.Aspects.FlowBehavior>. | `ref` or `out`. |
| <xref:PostSharp.Aspects.Advices.AwaiterAttribute> | An awaiter that the async method awaits after the advice, with <xref:PostSharp.Aspects.FlowBehavior.Yield?text=FlowBehavior.Yield>. | A type that implements <xref:System.Runtime.CompilerServices.INotifyCompletion>. | `ref` or `out`. |
| <xref:PostSharp.Aspects.Advices.AwaitedTaskAttribute> | The task that the async method awaits. | <xref:System.Threading.Tasks.Task>, <xref:System.IAsyncResult> or <xref:System.Object>. | By value. |
| <xref:PostSharp.Aspects.Advices.AwaitedMethodAttribute> | The method that the async method called to get the awaited object. | <xref:System.Reflection.MethodBase>, <xref:System.Reflection.MemberInfo> or <xref:System.Object>. | By value. |
| <xref:PostSharp.Aspects.Advices.CurrentTaskAttribute> | The task that the caller of the async method observes. `null` until the method yields. | <xref:System.Threading.Tasks.Task>, <xref:System.IAsyncResult> or <xref:System.Object>. | By value. |
| <xref:PostSharp.Aspects.Advices.AsyncCallIdAttribute> | The <xref:PostSharp.Aspects.AsyncCallId> that identifies the current execution of the async method. | <xref:PostSharp.Aspects.AsyncCallId>. | By value. |
| <xref:PostSharp.Aspects.Advices.DeclarationAttribute> | The reflection object of the target: <xref:System.Reflection.MethodBase>, <xref:PostSharp.Reflection.LocationInfo> or <xref:System.Type>, depending on the advice. It is not looked up on each call, except for generic targets. | The type of the reflection object, or a base type. | By value. |
| <xref:PostSharp.Aspects.Advices.DeclarationIdentifierAttribute> | The <xref:PostSharp.Reflection.DeclarationIdentifier> of the target. | <xref:PostSharp.Reflection.DeclarationIdentifier>. | By value. |
| <xref:PostSharp.Aspects.Advices.DeclarationNameAttribute> | The name of the target, which is a constant. With <xref:PostSharp.Aspects.Advices.DeclarationNameAttribute.IncludeTypeName>, the name is prefixed with the name of the declaring type. | <xref:System.String>. | By value. |
| <xref:PostSharp.Aspects.Advices.LocationValueAttribute> | The value of the field or property. | The type of the field or property. | `out` or `ref` in a get advice. Any in a set advice. |
| <xref:PostSharp.Aspects.Advices.BindingAttribute> | The binding object, which invokes the next node in the chain of interceptions of the target. | <xref:PostSharp.Aspects.IMethodBinding> in a method interception advice. <xref:PostSharp.Aspects.ILocationBinding`1> or <xref:PostSharp.Aspects.ILocationBinding> in a location advice. | By value. |


## Bindings by advice kind

The following advices accept bound parameters:

* Method boundary advices: <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodSuccessAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodExceptionAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodYieldAdvice> and <xref:PostSharp.Aspects.Advices.OnMethodResumeAdvice>
* Method interception advice: <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>
* Location interception advices: <xref:PostSharp.Aspects.Advices.OnLocationGetValueAdvice> and <xref:PostSharp.Aspects.Advices.OnLocationSetValueAdvice>
* <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice>, which runs once an object is constructed, whichever constructor ran

The other advices, such as <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice>, the event interception advices, <xref:PostSharp.Aspects.Advices.LocationValidationAdvice> and <xref:PostSharp.Aspects.Advices.OnInstanceLocationInitializedAdvice>, do not accept them. A binding attribute on one of their parameters causes the error LA0209 in <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice> and <xref:PostSharp.Aspects.Advices.OnInstanceLocationInitializedAdvice>, and the error LA0085 in the event interception advices and <xref:PostSharp.Aspects.Advices.LocationValidationAdvice>, whose signature is fixed.

The following table shows which binding attributes each advice accepts. In the column headings, Entry, Success, Exception, Exit, Yield and Resume are the method boundary advices, Invoke is <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, Location is the location interception advices, and Constructed is <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice>. Yes means that the attribute is accepted with the modes of the previous table. `out`, `ref` means that only these modes are accepted in this advice.

| Attribute | Entry | Success | Exception | Exit | Yield | Resume | Invoke | Location | Constructed |
|-----------|:-----:|:-------:|:---------:|:----:|:-----:|:------:|:------:|:--------:|:-----------:|
| `[Argument]` | Yes | Yes | Yes | Yes | Yes | Yes | Yes | | |
| `[Arguments]` | Yes | Yes | Yes | Yes | Yes | Yes | Yes | | |
| `[This]` | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| `[State]` | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| `[ReturnValue]` | `out`, `ref` | Yes | `out`, `ref` | Yes | | | `out`, `ref` | | |
| `[Exception]` | | | Yes | | | | | | |
| `[FlowBehavior]` | Yes | | Yes | | | Yes | | | |
| `[Awaiter]` | Yes | | | | | Yes | | | |
| `[AwaitedTask]`, `[AwaitedMethod]` | | | | | Yes | | | | |
| `[CurrentTask]`, `[AsyncCallId]` | Yes | Yes | Yes | Yes | Yes | Yes | | | |
| `[Declaration]`, `[DeclarationIdentifier]`, `[DeclarationName]` | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| `[LocationValue]` | | | | | | | | Yes | |
| `[Binding]` | | | | | | | Yes | Yes | |

A binding attribute in an advice that does not accept it causes the error LA0151. In <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice>, `[Argument]`, `[ReturnValue]`, `[LocationValue]` and `[FlowBehavior]` cause the error LA0224 instead. In the other advices, `[Arguments]` and `[FlowBehavior]` cause the warning LA0233, because the code builds but the value is ignored or not defined.

An advice can have at most one `[ReturnValue]` parameter and one `[Exception]` parameter. When an advice has several `[FlowBehavior]` parameters, only the last one is read, and PostSharp reports the warning LA0236. Several `[Awaiter]` parameters share one variable, and also cause the warning LA0236. The other attributes can be used several times in one advice, for instance to bind two arguments.


## Common scenarios

### Sharing state between advices

A parameter annotated with <xref:PostSharp.Aspects.Advices.StateAttribute> is bound to a variable that lives for one execution of the target method. Each execution has its own variable, and its initial value is the default value of its type.

Each variable is identified by a name, called a *slot*. The slot is the name of the parameter, or the second argument of the <xref:PostSharp.Aspects.Advices.StateAttribute> constructor. The advices of one [group](xref:pointcuts#grouping-advices) that bind the same slot share the variable, and all their parameters bound to this slot must have the same type. Advices of different groups, or of different aspects, never share a variable.

An `out` or `ref` parameter writes the variable. A parameter passed by value or as `in` reads it. In an async method or an iterator, the variable is a field of the state machine, so it keeps its value across `await` and `yield return` statements.

The only scope is <xref:PostSharp.Aspects.Advices.StateScope.MethodInvocation?text=StateScope.MethodInvocation>. To keep a value for the lifetime of an object, use a field of an instance-scoped aspect (see <xref:aspect-lifetime>).


### Reading and setting the return value

The meaning of a parameter annotated with <xref:PostSharp.Aspects.Advices.ReturnValueAttribute> depends on the advice:

* In <xref:PostSharp.Aspects.Advices.OnMethodSuccessAdvice> and <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice>, the advice reads the return value by value or as `in`, or replaces it with a `ref` parameter. When the method throws an exception, the exit advice reads the default value.
* In <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice> and <xref:PostSharp.Aspects.Advices.OnMethodExceptionAdvice>, the parameter is `out` or `ref`. It sets the value that the method returns when the advice sets the flow behavior to <xref:PostSharp.Aspects.FlowBehavior.Return?text=FlowBehavior.Return>. Without a `[ReturnValue]` parameter, the method returns <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue> when the advice has a <xref:PostSharp.Aspects.MethodExecutionArgs> parameter, and the default value of its return type otherwise.
* In <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, the parameter is `out` or `ref`, and the advice assigns the value that the intercepted method returns. An advice applied to a method that returns a value must have either a `[ReturnValue]` parameter or a <xref:PostSharp.Aspects.MethodInterceptionArgs> parameter (error LA0221).

In a method boundary advice applied semantically to an async method, or to a non-async method that returns a task, the return value is the result of the task (see <xref:semantic-advising>). When <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> disables semantic advising, and in an <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, the return value is the task or the enumerable itself. In a method boundary advice applied semantically, when the target method has no return value that the advice can access (it returns `void`, a task without a result or a reference, or it is an iterator), the parameter receives the default value, and a value written to it is ignored. Set <xref:PostSharp.Aspects.Advices.ReturnValueAttribute.IsRequired> to `true` to report an error instead.

The following aspect returns a cached value from the entry advice, and stores the value that the method returns in the success advice. The advices are generic, so the value is not boxed. <xref:PostSharp.Aspects.Advices.ReturnValueAttribute.IsRequired> makes the build fail on a method that returns no value, to which the aspect does not apply. `Cache<T>` is a class of your application.

```csharp
[PSerializable]
public sealed class CacheAttribute : MethodLevelAspect
{
    [OnMethodEntryAdvice]
    public static void OnEntry<T>( [Argument( 0 )] string key,
                                   [ReturnValue( IsRequired = true )] out T returnValue,
                                   [FlowBehavior] out FlowBehavior flowBehavior )
    {
        if ( Cache<T>.TryGetValue( key, out returnValue ) )
        {
            flowBehavior = FlowBehavior.Return;
        }
        else
        {
            flowBehavior = FlowBehavior.Default;
        }
    }

    [OnMethodSuccessAdvice]
    public static void OnSuccess<T>( [Argument( 0 )] string key, [ReturnValue] T returnValue )
    {
        Cache<T>.Set( key, returnValue );
    }
}
```

On an async method, the entry advice of this aspect causes the warning LA0154, because it has a `[FlowBehavior]` parameter but no `[Awaiter]` parameter.


### Handling exceptions

In <xref:PostSharp.Aspects.Advices.OnMethodExceptionAdvice>, a parameter annotated with <xref:PostSharp.Aspects.Advices.ExceptionAttribute> receives the exception thrown by the target method. With a `ref` or `out` parameter, the advice can replace the exception. The <xref:PostSharp.Aspects.FlowBehavior> value set through a `[FlowBehavior]` parameter decides what happens next:

* <xref:PostSharp.Aspects.FlowBehavior.Default?text=FlowBehavior.Default> and <xref:PostSharp.Aspects.FlowBehavior.RethrowException?text=FlowBehavior.RethrowException> rethrow the original exception, and ignore a replacement.
* <xref:PostSharp.Aspects.FlowBehavior.ThrowException?text=FlowBehavior.ThrowException> throws the exception that the `[Exception]` parameter holds. Without an `[Exception]` parameter, it throws <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> or the caught exception.
* <xref:PostSharp.Aspects.FlowBehavior.Return?text=FlowBehavior.Return> and <xref:PostSharp.Aspects.FlowBehavior.Continue?text=FlowBehavior.Continue> swallow the exception, and the method returns the value set through a `[ReturnValue]` parameter.

`FlowBehavior.ThrowException` throws the exception with a `throw` instruction, not a `rethrow` instruction, so the stack trace of the original exception is replaced.

The following advice wraps every exception into a `DataAccessException`:

```csharp
[OnMethodExceptionAdvice]
public static void OnException( [DeclarationName] string methodName,
                                [Exception] ref Exception exception,
                                [FlowBehavior] out FlowBehavior flowBehavior )
{
    exception = new DataAccessException( $"{methodName} failed.", exception );
    flowBehavior = FlowBehavior.ThrowException;
}
```

The `[Exception]` binding is not available in <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice>.


### Intercepting methods

In <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, the advice calls the intercepted method through the <xref:PostSharp.Aspects.IMethodBinding> received by a `[Binding]` parameter. The <xref:PostSharp.Aspects.IMethodBinding.Invoke*> method takes the instance and an <xref:PostSharp.Aspects.Arguments> object, which a `[Arguments]` parameter provides. The following aspect calls the method again, up to three times, when the method throws an <xref:System.IO.IOException>. For each target method, `T` is the return type of the method.

```csharp
[PSerializable]
public sealed class RetryAttribute : MethodLevelAspect
{
    [OnMethodInvokeAdvice]
    public static void OnInvoke<T>( [This] object instance,
                                    [Binding] IMethodBinding binding,
                                    [Arguments] Arguments arguments,
                                    [ReturnValue] out T returnValue )
    {
        for ( int attempt = 1; ; attempt++ )
        {
            try
            {
                returnValue = (T) binding.Invoke( ref instance, arguments );
                return;
            }
            catch ( IOException ) when ( attempt < 3 )
            {
            }
        }
    }
}
```

Apply this aspect only to methods that return a value and that are not async. An exception thrown after the first `await` of an async method is stored in the returned task, so this advice does not retry it.

An `[Argument]` parameter passed as `ref` or `out` in an interception advice does not change the argument that the intercepted method receives, and PostSharp reports the warning LA0235. To change an argument, change it in the <xref:PostSharp.Aspects.Arguments> object passed to the binding.


### Intercepting fields and properties

In <xref:PostSharp.Aspects.Advices.OnLocationGetValueAdvice> and <xref:PostSharp.Aspects.Advices.OnLocationSetValueAdvice>, a `[Binding]` parameter receives the <xref:PostSharp.Aspects.ILocationBinding`1> of the field or property, which gets or sets the underlying value. A `[LocationValue]` parameter is the value:

* In a get advice, the parameter is `out` or `ref`, and the advice assigns the value that the getter returns. A get advice must have either a `[LocationValue]` parameter or a <xref:PostSharp.Aspects.LocationInterceptionArgs> parameter (error LA0237).
* In a set advice, the parameter receives the value assigned to the field or property. The advice stores it by calling the binding.

On an instance field or property of a struct, the `instance` parameter receives a boxed copy, so the value is stored in the copy and lost. Apply such an aspect only to classes and static members, or keep the <xref:PostSharp.Aspects.LocationInterceptionArgs> signature for structs.

In a get advice, a `[LocationValue]` parameter passed by value compiles, but the getter then returns the default value.

```csharp
[PSerializable]
public sealed class TrimAttribute : LocationLevelAspect
{
    [OnLocationGetValueAdvice]
    public static void OnGetValue( [This] object instance,
                                   [Binding] ILocationBinding<string> binding,
                                   [LocationValue] out string value )
    {
        value = binding.GetValue( instance );
    }

    [OnLocationSetValueAdvice]
    public static void OnSetValue( [This] object instance,
                                   [Binding] ILocationBinding<string> binding,
                                   [LocationValue] string value )
    {
        binding.SetValue( instance, value?.Trim() );
    }
}
```


### Async methods

The method boundary advices of an async method can use the following bindings:

* `[AsyncCallId]` gives an identifier that is the same in all the advices of one execution of the method. It can correlate log records. On a method that is not async, the parameter receives the default value, unless <xref:PostSharp.Aspects.Advices.AsyncCallIdAttribute.IsRequired> is `true` (error LA0219).
* `[CurrentTask]` gives the task that the caller observes. The task exists only once the method has yielded, so the parameter is `null` in the entry advice, in the first yield advice, and in all the advices of an execution that completes without yielding. It is always `null` in a method that is not async, in a method implemented with runtime async (warning LA0240), and with a method builder other than the default builders of `Task` and `ValueTask`.
* `[AwaitedTask]` and `[AwaitedMethod]` give, in the yield advice, the task that the method awaits and the method that returned it.
* `[Awaiter]` and `[FlowBehavior]`, in the entry and resume advices, make the method await an awaiter of the aspect before it continues. The advice assigns the awaiter and sets the flow behavior to <xref:PostSharp.Aspects.FlowBehavior.Yield?text=FlowBehavior.Yield>. PostSharp Threading uses this mechanism to switch to another thread.

The `[State]` variables, the `ref` `[Argument]` parameters and the `[This]` instance of a struct are fields of the state machine, so the advices see the same values before and after an `await`.


### Mixing AdviceArgs and bound parameters

When an advice has both a <xref:PostSharp.Aspects.MethodExecutionArgs> parameter and a bound parameter that sets the same value, the bound parameter takes precedence, and PostSharp reports a warning:

* A `[ReturnValue]` parameter passed as `out` or `ref` takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue> (warning LA0243).
* A `[FlowBehavior]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> (warning LA0234).
* With <xref:PostSharp.Aspects.FlowBehavior.ThrowException?text=FlowBehavior.ThrowException>, an `[Exception]` parameter passed as `out` or `ref` takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> (warning LA0248).


## Generic advice methods

An advice method can be generic. For each target, PostSharp infers the type arguments from the values bound by `[Argument]`, `[Arguments]`, `[Binding]`, `[LocationValue]`, `[ReturnValue]` and `[This]`, with rules similar to the type inference of C#:

* A value of a value type is passed without boxing, and a `ref`, `out` or `in` parameter receives the location itself. In both cases, the type argument is exactly the type of the value.
* Otherwise, the value gives a lower bound. With several lower bounds, the type argument is the type to which all the values can be assigned. For instance, `OnEntry<T>( [Argument( 0 )] T a, [Argument( 1 )] T b )` applied to `M( string, object )` gives `object`.
* The type of the parameter can be a constructed type, such as `T[]`, `T?` or `IList<T>`. PostSharp then matches it with the type of the value, its base types and its interfaces.
* The constraints of the type parameters are checked with the inferred type arguments, including `unmanaged`, `new()` and constraints that refer to other type parameters.

When a type argument cannot be inferred, or when it violates a constraint, PostSharp reports the error LA0220, which gives the reason. A type parameter that no bound parameter uses is replaced by <xref:System.Object>, and PostSharp reports the warning LA0245.

The type of a parameter can also use the type parameters of a generic aspect class. They are replaced by the type arguments of the aspect instance.


## Selecting overloads with MatchPointcut

By default, an advice that does not fit a target causes an error. In a type-level aspect, the <xref:PostSharp.Aspects.Advices.MatchPointcut> pointcut skips these targets instead, so that several advices can handle the overloads of one method. See [Match pointcut](xref:pointcuts#match-pointcut).


## Limitations

The current version of advice parameter binding has the following limitations:

* A method interception advice cannot call the intercepted method, or read all the arguments, without an <xref:PostSharp.Aspects.Arguments> object, which `[Arguments]` allocates on each call.
* The event interception advices, <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice>, <xref:PostSharp.Aspects.Advices.LocationValidationAdvice> and <xref:PostSharp.Aspects.Advices.OnInstanceLocationInitializedAdvice> do not accept bound parameters.
* There is no binding for the value yielded by an iterator, which <xref:PostSharp.Aspects.MethodExecutionArgs.YieldValue> gives. <xref:PostSharp.Aspects.FlowBehavior.Yield?text=FlowBehavior.Yield> works only in the entry and resume advices of async methods.
* A `[State]` variable lives for one execution of the target method. There is no scope for an instance, a type or the application domain.


## Build messages

PostSharp checks the bound parameters when it builds the aspect, and again for each target. It reports all the errors of an aspect in one build. The following table lists the messages.

| Code | Severity | Meaning |
|------|----------|---------|
| LA0085 | Error | The advice has a fixed signature, such as an event interception advice or <xref:PostSharp.Aspects.Advices.LocationValidationAdvice>, and does not accept bound parameters. |
| LA0151 | Error | The binding attribute cannot be used in this advice. |
| LA0152 | Error | The `[Awaiter]` parameter does not implement <xref:System.Runtime.CompilerServices.INotifyCompletion>, or is not passed by reference. |
| LA0153 | Error | Two advices of one group have `[Awaiter]` parameters of different types. |
| LA0154 | Warning | An entry or resume advice of an async method has a `[FlowBehavior]` parameter but no `[Awaiter]` parameter, so it cannot use <xref:PostSharp.Aspects.FlowBehavior.Yield?text=FlowBehavior.Yield>. |
| LA0155 | Error | The `[FlowBehavior]` parameter is not of type <xref:PostSharp.Aspects.FlowBehavior>, or is not passed by reference. |
| LA0189 | Error | An `out` or `ref` `[Argument]` parameter is bound to an `in` argument. Use an `in` parameter. |
| LA0207 | Error | A parameter has more than one binding attribute. |
| LA0208 | Error | A parameter other than the first one has no binding attribute. |
| LA0209 | Error | The advice does not accept bound parameters. |
| LA0210 | Error | The binding attribute is not defined by PostSharp. |
| LA0211 | Error | The type of the parameter cannot receive the bound value. |
| LA0212 | Error | The parameter must be passed by reference (`out` or `ref`). |
| LA0213 | Error | The parameter cannot be passed by reference. |
| LA0214 | Error | The advice has two parameters with the same binding attribute, which allows only one. |
| LA0215 | Error | Two parameters bound to the same `[State]` slot have different types. |
| LA0216 | Error | The index of an `[Argument]` is negative. |
| LA0217 | Error | The type of the parameter is not compatible with the value bound on this target. |
| LA0218 | Error | `[This]` has <xref:PostSharp.Aspects.Advices.ThisAttribute.IsRequired> set to `true`, but the target is static. |
| LA0219 | Error | `[AsyncCallId]` has <xref:PostSharp.Aspects.Advices.AsyncCallIdAttribute.IsRequired> set to `true`, but the target is not an async method. |
| LA0220 | Error | A type argument of the generic advice cannot be inferred for this target. |
| LA0221 | Error | A method interception advice is applied to a method that returns a value, but it has neither a `[ReturnValue]` nor a <xref:PostSharp.Aspects.MethodInterceptionArgs> parameter. |
| LA0222 | Error | The address of an argument bound by reference cannot be loaded in this advice. |
| LA0223 | Error | `[This]` is passed by reference, but the target type is not a struct. |
| LA0224 | Error | `[Argument]`, `[ReturnValue]`, `[LocationValue]` or `[FlowBehavior]` is used in <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice>. |
| LA0225 | Error | The target method has no parameter at the index of the `[Argument]`. |
| LA0226 | Error | The target method has no parameter with the name of the `[Argument]`. |
| LA0227 | Error | The target method has no parameter of the type of the `[Argument]`. |
| LA0228 | Error | Several parameters of the target method have the type of the `[Argument]`. Select the argument by its index or name, or set <xref:PostSharp.Aspects.Advices.ArgumentAttribute.AmbiguityResolution>. |
| LA0229 | Error | The parameter must have exactly the type of the bound value, for instance <xref:System.String> for `[DeclarationName]`. |
| LA0233 | Warning | The binding attribute is not supported in this advice, so the value of the parameter is ignored or not defined. |
| LA0234 | Warning | The `[FlowBehavior]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior>. |
| LA0235 | Warning | A value written to an `out` or `ref` `[Argument]` of an interception advice does not reach the intercepted method. |
| LA0236 | Warning | The advice has several `[FlowBehavior]` or `[Awaiter]` parameters. |
| LA0237 | Error | A get advice has neither a `[LocationValue]` nor a <xref:PostSharp.Aspects.LocationInterceptionArgs> parameter. |
| LA0238 | Error | `[AwaitedTask]` or `[AwaitedMethod]` is used on a target that is not an async method. |
| LA0239 | Error | `[CurrentTask]` has <xref:PostSharp.Aspects.Advices.CurrentTaskAttribute.IsRequired> set to `true`, but the target cannot provide its task. |
| LA0240 | Warning | `[CurrentTask]` is always `null` because the C# compiler implemented the method with runtime async. |
| LA0241 | Error | `[ReturnValue]` has <xref:PostSharp.Aspects.Advices.ReturnValueAttribute.IsRequired> set to `true`, but the target has no return value that the advice can access. |
| LA0242 | Error | `[CurrentTask]` has <xref:PostSharp.Aspects.Advices.CurrentTaskAttribute.IsRequired> set to `true` in an advice that can run before the method yields. |
| LA0243 | Warning | The `out` or `ref` `[ReturnValue]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue>. |
| LA0244 | Error | `[CurrentTask]` has <xref:PostSharp.Aspects.Advices.CurrentTaskAttribute.IsRequired> set to `true`, but the entry advice of the group has an `[Awaiter]` parameter. |
| LA0245 | Warning | A type parameter of the generic advice is not used by any bound parameter, so its type argument is <xref:System.Object>. |
| LA0246 | Error | The type of a `[Declaration]` parameter cannot receive the reflection object of this kind of advice. |
| LA0247 | Error | The `[Exception]` parameter is not of type <xref:System.Exception>, or of type <xref:System.Object> by value. |
| LA0248 | Warning | The `out` or `ref` `[Exception]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.Exception>. |

To suppress a warning, see <xref:ignoring-warnings>.


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.AdviceParameterAttribute>
<br><xref:PostSharp.Aspects.Advices.StateAttribute>
<br><xref:PostSharp.Aspects.FlowBehavior>

**Other Resources**

<xref:member-level-aspects>
<br><xref:advices>
<br><xref:pointcuts>
<br><xref:modernizing-aspects>
<br><xref:advice-ordering>
<br><xref:semantic-advising>
