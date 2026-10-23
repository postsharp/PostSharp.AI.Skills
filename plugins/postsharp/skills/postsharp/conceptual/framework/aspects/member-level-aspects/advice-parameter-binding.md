---
uid: advice-parameter-binding
title: "Advice Parameters"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how an advice method receives the values it needs, such as arguments, the return value, the exception, a variable shared between advices or a member of the target type, as typed parameters annotated with binding attributes, and lists the bindings that each kind of advice accepts."
---
# Advice Parameters

An advice method receives the values that it needs from the target as parameters. Each parameter has a custom attribute, called a *binding attribute*, that selects the value: an argument, the instance, the return value, the exception, a variable shared by several advices, or a field, property or method of the target type. This mechanism is called *advice parameter binding*.

Bound parameters have the following benefits over a single <xref:PostSharp.Aspects.AdviceArgs> parameter, such as <xref:PostSharp.Aspects.MethodExecutionArgs>:

* The woven code computes only the values that the advice declares. No <xref:PostSharp.Aspects.AdviceArgs> object is allocated on each call.
* The parameters are typed. An argument of a value type is not boxed when the parameter has the type of the argument or is generic.
* PostSharp checks the signature of the advice when it builds the aspect, and again for each target. An advice that does not fit a target causes a build error instead of an exception at run time.

The binding attributes derive from <xref:PostSharp.Aspects.Advices.AdviceParameterAttribute> and are in the <xref:PostSharp.Aspects.Advices> namespace, except <xref:PostSharp.Aspects.Advices.ImportMemberAttribute>, which is also used on the fields of an aspect (see <xref:members>). For an introduction to advices and pointcuts, see <xref:advices>. To convert an existing aspect, see <xref:modernizing-aspects>.


## Example

The following aspect measures the duration of a method and writes it to the console with the value of the first argument. The entry advice starts a <xref:System.Diagnostics.Stopwatch> and stores it in a variable bound by <xref:PostSharp.Aspects.Advices.StateAttribute>. The advices have no pointcut, so they apply to the target method of the aspect and form one group (see [How it works](xref:member-level-aspects#how-it-works)). The `stopwatch` parameter of the exit advice is therefore bound to the same variable.

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/BindingStopwatch.cs#aspect)]


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
| <xref:PostSharp.Aspects.Advices.DeclarationNameAttribute> | The name of the target, which is a constant. With <xref:PostSharp.Aspects.Advices.DeclarationNameAttribute.IncludeTypeName>, the name is prefixed with the full name of the declaring type. | <xref:System.String>. | By value. |
| <xref:PostSharp.Aspects.Advices.LocationValueAttribute> | The value of the field or property. | The type of the field or property. | `out` or `ref` in a get advice. Any in a set advice. |
| <xref:PostSharp.Aspects.Advices.BindingAttribute> | The binding object, which invokes the next node in the chain of interceptions of the target. | <xref:PostSharp.Aspects.IMethodBinding> in a method interception advice. <xref:PostSharp.Aspects.ILocationBinding`1> or <xref:PostSharp.Aspects.ILocationBinding> in a location advice. | By value. |
| <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> | A field, a property or a method of the target type, selected by its name or by its type. See [Importing members of the target type](#importing-members-of-the-target-type). | The type of the field or property, <xref:PostSharp.Aspects.ILocationBinding`1>, <xref:PostSharp.Aspects.ILocationBinding>, a delegate type, <xref:PostSharp.Aspects.IMethodBinding>, <xref:PostSharp.Aspects.Advices.Property`1> or <xref:PostSharp.Aspects.Advices.Event`1>. | Any for a field. By value or `in` for a property. By value for the other types. |


## Bindings by advice kind

The following advices accept bound parameters:

* Method boundary advices: <xref:PostSharp.Aspects.Advices.OnMethodEntryAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodSuccessAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodExceptionAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice>, <xref:PostSharp.Aspects.Advices.OnMethodYieldAdvice> and <xref:PostSharp.Aspects.Advices.OnMethodResumeAdvice>
* Method interception advice: <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>
* Asynchronous method interception advice: <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice>, with `[This]` and `[ImportMember]` parameters only, passed by value, in a method that is not generic
* Location interception advices: <xref:PostSharp.Aspects.Advices.OnLocationGetValueAdvice> and <xref:PostSharp.Aspects.Advices.OnLocationSetValueAdvice>
* <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice>, which runs once an object is constructed, whichever constructor ran

The other advices, such as the event interception advices, <xref:PostSharp.Aspects.Advices.LocationValidationAdvice> and <xref:PostSharp.Aspects.Advices.OnInstanceLocationInitializedAdvice>, do not accept them. A binding attribute on one of their parameters causes the error <xref:la0209?text=LA0209> in <xref:PostSharp.Aspects.Advices.OnInstanceLocationInitializedAdvice>, and in <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice> for the bindings other than `[This]` and `[ImportMember]`, and for any binding in a generic method, and the error <xref:la0085?text=LA0085> in the event interception advices and <xref:PostSharp.Aspects.Advices.LocationValidationAdvice>, whose signature is fixed.

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
| `[ImportMember]` | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |

A binding attribute in an advice that does not accept it causes the error <xref:la0151?text=LA0151>. In <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice>, `[Argument]`, `[ReturnValue]`, `[LocationValue]` and `[FlowBehavior]` cause the error <xref:la0224?text=LA0224> instead. In the other advices, `[Arguments]` and `[FlowBehavior]` cause the warning <xref:la0233?text=LA0233>, because the code builds but the value is ignored or not defined.

An advice can have at most one `[ReturnValue]` parameter and one `[Exception]` parameter. When an advice has several `[FlowBehavior]` parameters, only the last one is read, and PostSharp reports the warning <xref:la0236?text=LA0236>. Several `[Awaiter]` parameters share one variable, and also cause the warning <xref:la0236?text=LA0236>. The other attributes can be used several times in one advice, for instance to bind two arguments.


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
* In <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, the parameter is `out` or `ref`, and the advice assigns the value that the intercepted method returns. An advice applied to a method that returns a value must have either a `[ReturnValue]` parameter or a <xref:PostSharp.Aspects.MethodInterceptionArgs> parameter (error <xref:la0221?text=LA0221>).

In a method boundary advice applied semantically to an async method, or to a non-async method that returns a task, the return value is the result of the task (see <xref:semantic-advising>). When <xref:PostSharp.Aspects.Advices.OnMethodBoundaryAdvice.SemanticallyAdvisedMethodKinds> disables semantic advising, and in an <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, the return value is the task or the enumerable itself. In a method boundary advice applied semantically, when the target method has no return value that the advice can access (it returns `void`, a task without a result or a reference, or it is an iterator), the parameter receives the default value, and a value written to it is ignored. Set <xref:PostSharp.Aspects.Advices.ReturnValueAttribute.IsRequired> to `true` to report an error instead.

The following aspect returns a cached value from the entry advice, and stores the value that the method returns in the success advice. The advices are generic, so the value is not boxed. <xref:PostSharp.Aspects.Advices.ReturnValueAttribute.IsRequired> makes the build fail on a method that returns no value, to which the aspect does not apply. `Cache<T>` is a class of your application.

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/BindingCache.cs#aspect)]

When the aspect is applied to a `GetStock` method, calling the method twice with the same argument executes it only once:

[!code-text[](../../../../code/samples/framework/aspects/member-level-aspects/BindingCache.txt)]

On an async method, the entry advice of this aspect causes the warning <xref:la0154?text=LA0154>, because it has a `[FlowBehavior]` parameter but no `[Awaiter]` parameter.


### Handling exceptions

In <xref:PostSharp.Aspects.Advices.OnMethodExceptionAdvice>, a parameter annotated with <xref:PostSharp.Aspects.Advices.ExceptionAttribute> receives the exception thrown by the target method. With a `ref` or `out` parameter, the advice can replace the exception. The <xref:PostSharp.Aspects.FlowBehavior> value set through a `[FlowBehavior]` parameter decides what happens next:

* <xref:PostSharp.Aspects.FlowBehavior.Default?text=FlowBehavior.Default> and <xref:PostSharp.Aspects.FlowBehavior.RethrowException?text=FlowBehavior.RethrowException> rethrow the original exception, and ignore a replacement.
* <xref:PostSharp.Aspects.FlowBehavior.ThrowException?text=FlowBehavior.ThrowException> throws the exception that the `[Exception]` parameter holds. Without an `[Exception]` parameter, it throws <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> or the caught exception.
* <xref:PostSharp.Aspects.FlowBehavior.Return?text=FlowBehavior.Return> and <xref:PostSharp.Aspects.FlowBehavior.Continue?text=FlowBehavior.Continue> swallow the exception, and the method returns the value set through a `[ReturnValue]` parameter.

`FlowBehavior.ThrowException` throws the exception with a `throw` instruction, not a `rethrow` instruction, so the stack trace of the original exception is replaced.

The following advice wraps every exception into a `DataAccessException`:

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/BindingException.cs#advice)]

The `[Exception]` binding is not available in <xref:PostSharp.Aspects.Advices.OnMethodExitAdvice>.


### Intercepting methods

In <xref:PostSharp.Aspects.Advices.OnMethodInvokeAdvice>, the advice calls the intercepted method through the <xref:PostSharp.Aspects.IMethodBinding> received by a `[Binding]` parameter. The <xref:PostSharp.Aspects.IMethodBinding.Invoke*> method takes the instance and an <xref:PostSharp.Aspects.Arguments> object, which a `[Arguments]` parameter provides. The following aspect calls the method again, up to three times, when the method throws an <xref:System.IO.IOException>. For each target method, `T` is the return type of the method.

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/BindingRetry.cs#aspect)]

When the aspect is applied to a method that fails twice with an <xref:System.IO.IOException>, the third call succeeds:

[!code-text[](../../../../code/samples/framework/aspects/member-level-aspects/BindingRetry.txt)]

Apply this aspect only to methods that return a value and that are not async. An exception thrown after the first `await` of an async method is stored in the returned task, so this advice does not retry it.

An `[Argument]` parameter passed as `ref` or `out` in an interception advice does not change the argument that the intercepted method receives, and PostSharp reports the warning <xref:la0235?text=LA0235>. To change an argument, change it in the <xref:PostSharp.Aspects.Arguments> object passed to the binding.


### Intercepting fields and properties

In <xref:PostSharp.Aspects.Advices.OnLocationGetValueAdvice> and <xref:PostSharp.Aspects.Advices.OnLocationSetValueAdvice>, a `[Binding]` parameter receives the <xref:PostSharp.Aspects.ILocationBinding`1> of the field or property, which gets or sets the underlying value. A `[LocationValue]` parameter is the value:

* In a get advice, the parameter is `out` or `ref`, and the advice assigns the value that the getter returns. A get advice must have either a `[LocationValue]` parameter or a <xref:PostSharp.Aspects.LocationInterceptionArgs> parameter (error <xref:la0237?text=LA0237>).
* In a set advice, the parameter receives the value assigned to the field or property. The advice stores it by calling the binding.

On an instance field or property of a struct, the `instance` parameter receives a boxed copy, so the value is stored in the copy and lost. Apply such an aspect only to classes and static members, or keep the <xref:PostSharp.Aspects.LocationInterceptionArgs> signature for structs.

In a get advice, a `[LocationValue]` parameter passed by value compiles, but the getter then returns the default value.

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/BindingTrim.cs#aspect)]


### Importing members of the target type

A parameter annotated with <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> receives a field, a property or a method of the target type. The target type is the type to which the aspect is applied, or the type that declares the member to which it is applied. The aspect can be of any kind, including a statically scoped aspect such as a <xref:PostSharp.Aspects.MethodLevelAspect>. An aspect field annotated with <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> requires an <xref:PostSharp.Aspects.InstanceLevelAspect> (see <xref:members>), but an advice parameter does not.

The type and the mode of the parameter select what the advice receives:

* A parameter of the type of a field receives the field. By value or as `in`, it reads a copy of the value. As `ref` or `out`, it receives a reference to the field, so the advice can change the field. A `ref` or `out` parameter requires a field that is not read-only, or a property that returns a reference that is not read-only.
* A parameter of the type of a property, by value or as `in`, receives the value that the getter returns when the advice is called.
* A parameter of type <xref:PostSharp.Aspects.ILocationBinding`1> or <xref:PostSharp.Aspects.ILocationBinding> receives an object that reads and writes the field or property on the instance passed to its methods, typically received by a `[This]` parameter. There is one binding object per member and target type.
* A parameter of a delegate type, with a member name, receives an *open delegate* of a method. For an instance method, the first parameter of the delegate receives the instance, and the other parameters receive the arguments. The delegate type selects the overload, as when C# converts a method group to a delegate, and a virtual method is called with virtual dispatch. The delegate is created once per target type, so calling it does not allocate. For a struct, see the note below.
* A parameter of type <xref:PostSharp.Aspects.IMethodBinding>, with a member name, receives an object that calls the method on the instance passed to <xref:PostSharp.Aspects.IMethodBinding.Invoke*>, with the arguments of an <xref:PostSharp.Aspects.Arguments> object. It selects the method by its name only, so a method that has overloads must be imported as a delegate. The method must not be generic, and must not return by reference.
* A parameter of type <xref:PostSharp.Aspects.IEventBinding>, with a member name, receives an object that adds and removes handlers of the event on the instance passed to its methods.
* In an instance advice of an instance-scoped aspect (see <xref:aspect-lifetime>), a parameter of type <xref:PostSharp.Aspects.Advices.Property`1> or <xref:PostSharp.Aspects.Advices.Event`1>, with a member name, receives delegates bound to the target object, as an aspect field of these types does.

When the attribute has several member names, PostSharp tries them in order, and selects the first member whose kind and type the parameter accepts. Without a name, it selects a field or a property by its type, as `[Argument]` selects an argument: a parameter passed by value accepts a member whose type can be assigned to it without a conversion other than boxing, and a parameter passed by reference, or a location binding, requires exactly the type of the member. When several members match, the build fails, and you must specify the name. A method is always selected by its name.

> [!NOTE]
> To import a method of a struct as a delegate, declare a delegate type of your own: its first parameter must be `in`, or `ref` for a method that is not `readonly`.

The candidates are the members of the target type and the accessible members of its base types, nearest first. Indexers, constants and the fields generated by the compiler are not candidates. The members introduced with <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute> are not candidates either, because PostSharp selects the member before the introductions. The fields that the aspect requests with <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance> are candidates, and they win over the other members (see [Requesting fields](xref:advice-provider#requesting-fields)).

When no member matches, a parameter passed by value receives `null` or the default value, and a `ref` or `out` parameter receives a reference to a temporary variable. Set <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> to `true` to make the build fail instead. On a static target, a parameter passed by value receives the default value of an instance member, and a parameter passed by reference to an instance field causes an error.

The following aspect is a <xref:PostSharp.Aspects.MethodLevelAspect>, which is statically scoped. Its advice increments the `callCount` field of the target object through a `ref` parameter, reads the `Name` property, and calls the private `OnCalled` method through an open delegate. The advice is generic: `T` is the type of the target object, and the delegate type `Action<T, int>` selects the method `OnCalled( int )`.

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/BindingImportMember.cs#aspect)]

The aspect is applied to the `Print` method of the following class:

[!code-csharp[](../../../../code/samples/framework/aspects/member-level-aspects/BindingImportMember.cs#target)]

Each object has its own counter, which the class can also read and reset:

[!code-text[](../../../../code/samples/framework/aspects/member-level-aspects/BindingImportMember.txt)]

When another aspect turns the imported field into a property, for instance a <xref:PostSharp.Aspects.LocationInterceptionAspect>, the order of the two aspects decides what the parameter receives. An aspect ordered before the other one receives the field itself. An aspect ordered after it receives the property: the values go through the accessors, and a `ref` or `out` parameter causes an error. Order the aspects with aspect dependencies (see <xref:aspect-dependencies>).

For a method, a property or an event imported as a delegate, an <xref:PostSharp.Aspects.IMethodBinding>, a <xref:PostSharp.Aspects.Advices.Property`1> or an <xref:PostSharp.Aspects.Advices.Event`1>, <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.Order> has the meaning that it has on an aspect field. With <xref:PostSharp.Aspects.Advices.ImportMemberOrder.BeforeIntroductions?text=ImportMemberOrder.BeforeIntroductions>, the parameter calls the method as it was before the aspect introduced or overrode it, without virtual dispatch.


### Async methods

The method boundary advices of an async method can use the following bindings:

* `[AsyncCallId]` gives an identifier that is the same in all the advices of one execution of the method. It can correlate log records. On a method that is not async, the parameter receives the default value, unless <xref:PostSharp.Aspects.Advices.AsyncCallIdAttribute.IsRequired> is `true` (error <xref:la0219?text=LA0219>).
* `[CurrentTask]` gives the task that the caller observes. The task exists only once the method has yielded, so the parameter is `null` in the entry advice, in the first yield advice, and in all the advices of an execution that completes without yielding. It is always `null` in a method that is not async, in a method implemented with runtime async (warning <xref:la0240?text=LA0240>), and with a method builder other than the default builders of `Task` and `ValueTask`.
* `[AwaitedTask]` and `[AwaitedMethod]` give, in the yield advice, the task that the method awaits and the method that returned it.
* `[Awaiter]` and `[FlowBehavior]`, in the entry and resume advices, make the method await an awaiter of the aspect before it continues. The advice assigns the awaiter and sets the flow behavior to <xref:PostSharp.Aspects.FlowBehavior.Yield?text=FlowBehavior.Yield>. PostSharp Threading uses this mechanism to switch to another thread.

The `[State]` variables, the `ref` `[Argument]` parameters and the `[This]` instance of a struct are fields of the state machine, so the advices see the same values before and after an `await`.


### Mixing AdviceArgs and bound parameters

When an advice has both a <xref:PostSharp.Aspects.MethodExecutionArgs> parameter and a bound parameter that sets the same value, the bound parameter takes precedence, and PostSharp reports a warning:

* A `[ReturnValue]` parameter passed as `out` or `ref` takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue> (warning <xref:la0243?text=LA0243>).
* A `[FlowBehavior]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> (warning <xref:la0234?text=LA0234>).
* With <xref:PostSharp.Aspects.FlowBehavior.ThrowException?text=FlowBehavior.ThrowException>, an `[Exception]` parameter passed as `out` or `ref` takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> (warning <xref:la0248?text=LA0248>).


## Generic advice methods

An advice method can be generic. For each target, PostSharp infers the type arguments from the values bound by `[Argument]`, `[Arguments]`, `[Binding]`, `[LocationValue]`, `[ReturnValue]` and `[This]`, with rules similar to the type inference of C#:

* A value of a value type is passed without boxing, and a `ref`, `out` or `in` parameter receives the location itself. In both cases, the type argument is exactly the type of the value.
* Otherwise, the value gives a lower bound. With several lower bounds, the type argument is the type to which all the values can be assigned. For instance, `OnEntry<T>( [Argument( 0 )] T a, [Argument( 1 )] T b )` applied to `M( string, object )` gives `object`.
* The type of the parameter can be a constructed type, such as `T[]`, `T?` or `IList<T>`. PostSharp then matches it with the type of the value, its base types and its interfaces.
* The constraints of the type parameters are checked with the inferred type arguments, including `unmanaged`, `new()` and constraints that refer to other type parameters.

When a type argument cannot be inferred, or when it violates a constraint, PostSharp reports the error <xref:la0220?text=LA0220>, which gives the reason. A type parameter that no bound parameter uses is replaced by <xref:System.Object>, and PostSharp reports the warning <xref:la0245?text=LA0245>.

The type of a parameter can also use the type parameters of a generic aspect class. They are replaced by the type arguments of the aspect instance.


## Selecting overloads with MatchPointcut

By default, an advice that does not fit a target causes an error. In a type-level aspect, the <xref:PostSharp.Aspects.Advices.MatchPointcut> pointcut skips these targets instead, so that several advices can handle the overloads of one method. See [Match pointcut](xref:pointcuts#match-pointcut).


## Limitations

The current version of advice parameter binding has the following limitations:

* A method interception advice cannot call the intercepted method, or read all the arguments, without an <xref:PostSharp.Aspects.Arguments> object, which `[Arguments]` allocates on each call.
* The event interception advices, <xref:PostSharp.Aspects.Advices.LocationValidationAdvice> and <xref:PostSharp.Aspects.Advices.OnInstanceLocationInitializedAdvice> do not accept bound parameters. <xref:PostSharp.Aspects.Advices.OnMethodInvokeAsyncAdvice> accepts only `[This]` and `[ImportMember]`, by value, in a method that is not generic.
* There is no binding for the value yielded by an iterator, which <xref:PostSharp.Aspects.MethodExecutionArgs.YieldValue> gives. <xref:PostSharp.Aspects.FlowBehavior.Yield?text=FlowBehavior.Yield> works only in the entry and resume advices of async methods.
* A `[State]` variable lives for one execution of the target method. There is no scope for an instance, a type or the application domain.


## Build messages

PostSharp checks the bound parameters when it builds the aspect, and again for each target. It reports all the errors of an aspect in one build. The following table lists the messages.

| Code | Severity | Meaning |
|------|----------|---------|
| <xref:la0085?text=LA0085> | Error | The advice has a fixed signature, such as an event interception advice or <xref:PostSharp.Aspects.Advices.LocationValidationAdvice>, and does not accept bound parameters. |
| <xref:la0151?text=LA0151> | Error | The binding attribute cannot be used in this advice. |
| <xref:la0152?text=LA0152> | Error | The `[Awaiter]` parameter does not implement <xref:System.Runtime.CompilerServices.INotifyCompletion>, or is not passed by reference. |
| <xref:la0153?text=LA0153> | Error | Two advices of one group have `[Awaiter]` parameters of different types. |
| <xref:la0154?text=LA0154> | Warning | An entry or resume advice of an async method has a `[FlowBehavior]` parameter but no `[Awaiter]` parameter, so it cannot use <xref:PostSharp.Aspects.FlowBehavior.Yield?text=FlowBehavior.Yield>. |
| <xref:la0155?text=LA0155> | Error | The `[FlowBehavior]` parameter is not of type <xref:PostSharp.Aspects.FlowBehavior>, or is not passed by reference. |
| <xref:la0189?text=LA0189> | Error | An `out` or `ref` `[Argument]` parameter is bound to an `in` argument. Use an `in` parameter. |
| <xref:la0207?text=LA0207> | Error | A parameter has more than one binding attribute. |
| <xref:la0208?text=LA0208> | Error | A parameter other than the first one has no binding attribute. |
| <xref:la0209?text=LA0209> | Error | The advice does not accept bound parameters. |
| <xref:la0210?text=LA0210> | Error | The binding attribute is not defined by PostSharp. |
| <xref:la0211?text=LA0211> | Error | The type of the parameter cannot receive the bound value. |
| <xref:la0212?text=LA0212> | Error | The parameter must be passed by reference (`out` or `ref`). |
| <xref:la0213?text=LA0213> | Error | The parameter cannot be passed by reference. |
| <xref:la0214?text=LA0214> | Error | The advice has two parameters with the same binding attribute, which allows only one. |
| <xref:la0215?text=LA0215> | Error | Two parameters bound to the same `[State]` slot have different types. |
| <xref:la0216?text=LA0216> | Error | The index of an `[Argument]` is negative. |
| <xref:la0217?text=LA0217> | Error | The type of the parameter is not compatible with the value bound on this target. |
| <xref:la0218?text=LA0218> | Error | `[This]` has <xref:PostSharp.Aspects.Advices.ThisAttribute.IsRequired> set to `true`, but the target is static. |
| <xref:la0219?text=LA0219> | Error | `[AsyncCallId]` has <xref:PostSharp.Aspects.Advices.AsyncCallIdAttribute.IsRequired> set to `true`, but the target is not an async method. |
| <xref:la0220?text=LA0220> | Error | A type argument of the generic advice cannot be inferred for this target. |
| <xref:la0221?text=LA0221> | Error | A method interception advice is applied to a method that returns a value, but it has neither a `[ReturnValue]` nor a <xref:PostSharp.Aspects.MethodInterceptionArgs> parameter. |
| <xref:la0222?text=LA0222> | Error | The address of an argument bound by reference cannot be loaded in this advice. |
| <xref:la0223?text=LA0223> | Error | `[This]` is passed by reference, but the target type is not a struct. |
| <xref:la0224?text=LA0224> | Error | `[Argument]`, `[ReturnValue]`, `[LocationValue]` or `[FlowBehavior]` is used in <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice>. |
| <xref:la0225?text=LA0225> | Error | The target method has no parameter at the index of the `[Argument]`. |
| <xref:la0226?text=LA0226> | Error | The target method has no parameter with the name of the `[Argument]`. |
| <xref:la0227?text=LA0227> | Error | The target method has no parameter of the type of the `[Argument]`. |
| <xref:la0228?text=LA0228> | Error | Several parameters of the target method have the type of the `[Argument]`. Select the argument by its index or name, or set <xref:PostSharp.Aspects.Advices.ArgumentAttribute.AmbiguityResolution>. |
| <xref:la0229?text=LA0229> | Error | The parameter must have exactly the type of the bound value, for instance <xref:System.String> for `[DeclarationName]`. |
| <xref:la0233?text=LA0233> | Warning | The binding attribute is not supported in this advice, so the value of the parameter is ignored or not defined. |
| <xref:la0234?text=LA0234> | Warning | The `[FlowBehavior]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior>. |
| <xref:la0235?text=LA0235> | Warning | A value written to an `out` or `ref` `[Argument]` of an interception advice does not reach the intercepted method. |
| <xref:la0236?text=LA0236> | Warning | The advice has several `[FlowBehavior]` or `[Awaiter]` parameters. |
| <xref:la0237?text=LA0237> | Error | A get advice has neither a `[LocationValue]` nor a <xref:PostSharp.Aspects.LocationInterceptionArgs> parameter. |
| <xref:la0238?text=LA0238> | Error | `[AwaitedTask]` or `[AwaitedMethod]` is used on a target that is not an async method. |
| <xref:la0239?text=LA0239> | Error | `[CurrentTask]` has <xref:PostSharp.Aspects.Advices.CurrentTaskAttribute.IsRequired> set to `true`, but the target cannot provide its task. |
| <xref:la0240?text=LA0240> | Warning | `[CurrentTask]` is always `null` because the C# compiler implemented the method with runtime async. |
| <xref:la0241?text=LA0241> | Error | `[ReturnValue]` has <xref:PostSharp.Aspects.Advices.ReturnValueAttribute.IsRequired> set to `true`, but the target has no return value that the advice can access. |
| <xref:la0242?text=LA0242> | Error | `[CurrentTask]` has <xref:PostSharp.Aspects.Advices.CurrentTaskAttribute.IsRequired> set to `true` in an advice that can run before the method yields. |
| <xref:la0243?text=LA0243> | Warning | The `out` or `ref` `[ReturnValue]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue>. |
| <xref:la0244?text=LA0244> | Error | `[CurrentTask]` has <xref:PostSharp.Aspects.Advices.CurrentTaskAttribute.IsRequired> set to `true`, but the entry advice of the group has an `[Awaiter]` parameter. |
| <xref:la0245?text=LA0245> | Warning | A type parameter of the generic advice is not used by any bound parameter, so its type argument is <xref:System.Object>. |
| <xref:la0246?text=LA0246> | Error | The type of a `[Declaration]` parameter cannot receive the reflection object of this kind of advice. |
| <xref:la0247?text=LA0247> | Error | The `[Exception]` parameter is not of type <xref:System.Exception>, or of type <xref:System.Object> by value. |
| <xref:la0248?text=LA0248> | Warning | The `out` or `ref` `[Exception]` parameter takes precedence over <xref:PostSharp.Aspects.MethodExecutionArgs.Exception>. |
| <xref:la0260?text=LA0260> | Error | An `[ImportMember]` parameter without a name has <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> set to `true`, but no field or property of the target type has a type that the parameter can receive. |
| <xref:la0261?text=LA0261> | Error | An `[ImportMember]` parameter is passed as `out` or `ref` to a field that an aspect ordered before it turns into a property. |
| <xref:la0262?text=LA0262> | Error | An `[ImportMember]` parameter is passed as `out` or `ref` to a field of a struct in an async method or an iterator, which works on a copy of the struct. |
| <xref:la0263?text=LA0263> | Error | An `[ImportMember]` delegate parameter has <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> set to `true`, but no method of the name converts to the delegate type. |
| <xref:la0264?text=LA0264> | Error | Several methods of the name convert to the delegate type of an `[ImportMember]` parameter. |
| <xref:la0265?text=LA0265> | Error | An `[ImportMember]` parameter of type <xref:PostSharp.Aspects.IMethodBinding> has <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> set to `true`, but the target type has no method of the name that is not generic and does not return by reference. |
| <xref:la0266?text=LA0266> | Error | An `[ImportMember]` parameter of type <xref:PostSharp.Aspects.IMethodBinding> names a method that has several overloads. Import the method as a delegate. |
| <xref:la0267?text=LA0267> | Error | An `[ImportMember]` parameter of type <xref:PostSharp.Aspects.Advices.Property`1> or <xref:PostSharp.Aspects.Advices.Event`1> is not in an instance advice of an instance-scoped aspect. |
| <xref:la0277?text=LA0277> | Error | The delegate type of an `[ImportMember]` parameter passes a struct as `ref` to a `readonly` method. Declare the first parameter of the delegate type as `in`. |
| <xref:la0280?text=LA0280> | Error | The type of an `[ImportMember]` parameter is not supported: <xref:PostSharp.Aspects.Advices.Property`2>. |
| <xref:la0281?text=LA0281> | Error | An `[ImportMember]` parameter with a name has <xref:PostSharp.Aspects.Advices.ImportMemberAttribute.IsRequired> set to `true`, but the target type has no accessible field or property of this name that the parameter can receive. |
| <xref:la0282?text=LA0282> | Error | An `[ImportMember]` parameter is passed by reference to an instance field, but the target is static. |
| <xref:la0283?text=LA0283> | Error | An `[ImportMember]` parameter without a name matches several members of the target type. Specify the name. |
| <xref:la0284?text=LA0284> | Error | An `[ImportMember]` parameter of this type, such as <xref:PostSharp.Aspects.ILocationBinding>, imports a member by name only. Specify the name. |

To suppress a warning, see <xref:ignoring-warnings>.


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.AdviceParameterAttribute>
<br><xref:PostSharp.Aspects.Advices.ImportMemberAttribute>
<br><xref:PostSharp.Aspects.Advices.StateAttribute>
<br><xref:PostSharp.Aspects.FlowBehavior>

**Other Resources**

<xref:member-level-aspects>
<br><xref:advices>
<br><xref:pointcuts>
<br><xref:members>
<br><xref:advice-provider>
<br><xref:modernizing-aspects>
<br><xref:advice-ordering>
<br><xref:semantic-advising>
