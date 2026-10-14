---
uid: method-interception
redirect_from:
  - custompatterns/aspects/method-interception
title: "Intercepting Methods"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document outlines how to use PostSharp's MethodInterceptionAspect class to intercept method invocations and replace them with custom hooks. It also provides examples and instructions on how to create an aspect that retries a method call on exception."
---
# Intercepting Methods

It is often useful to be able to intercept the invocation of a method and invoke your own hook in its place. Common use cases for this capability include dispatching the method execution to a different thread, asynchronously executing the method at a later time, and retrying the method call when an exception is thrown.

PostSharp addresses these needs with the <xref:PostSharp.Aspects.MethodInterceptionAspect> aspect class, which intercepts the invocation of a method before the method is executed. It also allows you to invoke the original method and access its arguments and return value. 

This article covers method *interception*. For another approach to injecting behaviors into methods, see <xref:method-decorator>. 


## Intercepting a method call


### Creating an aspect that retries a method call on exception

1. Add a reference to the *PostSharp* package to your project. 


2. Create an aspect class and derive it from <xref:PostSharp.Aspects.MethodInterceptionAspect>. Annotate the class with the [<xref:PostSharp.Serialization.PSerializableAttribute>] custom attribute. 


3. Override the <xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvoke(PostSharp.Aspects.MethodInterceptionArgs)> method and implement the interception logic. Call `args.Proceed()` to invoke the intercepted method. 

    > [!NOTE]
    > Calling `base.OnInvoke()` is equivalent to calling `args.Proceed()`. 


4. Add the aspect to one or more methods. Since <xref:PostSharp.Aspects.MethodInterceptionAspect> derives from the <xref:System.Attribute> class, you can add the aspect custom attribute to the methods you need. If you need to add the aspect to more methods (for instance all public methods in a namespace), you can learn about more advanced techniques in <xref:applying-aspects>. 



### Example

Consider the following `CustomerService` class, which has methods to load and save customer entities and relies on calls to a database or a web service. 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionService.cs#example)]

Occasionally, the connection to the underlying store may become unreliable, and the application user is presented with an error message. To improve the user experience, you may want to retry the failing operation several times before displaying the error message. In the following code, we create a method interception class that can be applied to repository methods and retries the invocation whenever the original method throws an exception.

The complete aspect code is as follows:

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionRetry.cs#aspect)]

Apply the `[RetryOnException]` custom attributes to all methods where the behavior is needed. 

In the following snippet, this aspect is applied to the `CustomerService.Save` method: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionRetry.cs#service)]

Whenever the `CustomerService.Save` method is invoked, the `RetryOnExceptionAttribute.OnInvoke` method is called instead. The aspect method will invoke the original method and retry if necessary. 

When the database call fails twice before it succeeds, the program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionRetry.txt)]


## Accessing the current execution context

As illustrated in the example above, you can access information about the method being intercepted from the property <xref:PostSharp.Aspects.MethodInterceptionArgs.Method>, which gives you the <xref:System.Reflection.MethodBase> of the method that has been intercepted. This object gives you access to parameters, return type, declaring type, and other characteristics. In case of generic methods or generic types, <xref:PostSharp.Aspects.MethodInterceptionArgs.Method> gives you the proper generic method instance, so you can use this object to get generic parameters. 

The <xref:PostSharp.Aspects.MethodInterceptionArgs> parameter gives you access to other pieces of information regarding the current execution context. 

| Property | Description |
|----------|-------------|
| <xref:PostSharp.Aspects.MethodInterceptionArgs.Method> | The method being executed (in case of generic methods, this property is set to the proper generic instance of the method). |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.Arguments> | The arguments passed to the method. If you modify the arguments and call `args.Proceed()`, the intercepted method will be invoked with the modified arguments.  |
| <xref:PostSharp.Aspects.AdviceArgs.Instance> | The object on which the method is being executed, that is, the value of the `this` keyword.  |
| <xref:PostSharp.Aspects.MethodInterceptionArgs.ReturnValue> | The return value of the method. This property is populated after the aspect calls `args.Proceed`. The aspect can then modify the value of this property if it needs to return a different value than the one returned by the intercepted method.  |


## Intercepting methods returning a Task, including async methods

The <xref:System.Threading.Tasks.Task> class in .NET represents operations that can execute asynchronously. Whenever you want to intercept a method that returns a <xref:System.Threading.Tasks.Task>, you have two options for defining the target of the interception: 

* Intercepting the logic that creates and returns a new <xref:System.Threading.Tasks.Task>. The logic of the asynchronous operation represented by the <xref:System.Threading.Tasks.Task> is not intercepted, and the status of <xref:System.Threading.Tasks.Task>, return value and thrown exception are not handled by an aspect. This is what happens when you intercept a <xref:System.Threading.Tasks.Task>-returning method with an aspect implementing only <xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvoke(PostSharp.Aspects.MethodInterceptionArgs)>. 

* Intercepting both the logic that instantiates the <xref:System.Threading.Tasks.Task> and the logic of the <xref:System.Threading.Tasks.Task>. In this case, you intercept the asynchronous operation represented by the task. You can await the completion of the task, and you can handle the return value of the task and thrown exception inside the aspect. This interception mode is called *semantic advising*. 

To intercept the whole Task logic, your aspect must implement the <xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvokeAsync(PostSharp.Aspects.MethodInterceptionArgs)> method. Your implementation can call `args.ProceedAsync()` instead of `args.Proceed()` to invoke the intercepted method and execute the intercepted task. 

When an aspect implements both <xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvoke(PostSharp.Aspects.MethodInterceptionArgs)> and <xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvokeAsync(PostSharp.Aspects.MethodInterceptionArgs)>, PostSharp automatically selects the proper method when the target method returns a <xref:System.Threading.Tasks.Task>. You can change this behavior by changing the value of the <xref:PostSharp.Aspects.MethodInterceptionAspect.SemanticallyAdvisedMethodKinds> aspect property. See <xref:semantic-advising> for details regarding this property. 

In this article, we will demonstrate how to control the semantic behavior of the <xref:PostSharp.Aspects.MethodInterceptionAspect> aspect when it is applied to a method returning a task or to an async method. 


### Example

To demonstrate when the semantic approach to method interception can be useful, we extend the `RetryOnExceptionAttribute` aspect created in the previous example. 

The previous version of `RetryOnExceptionAttribute` aspect intercepts the original method, but it does not intercept the async task returned by the method. As a result, the aspect cannot catch the exception thrown by the async task, and the exception reaches the caller. To handle async methods properly, we need to add an implementation of the <xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvokeAsync(PostSharp.Aspects.MethodInterceptionArgs)> method to the `RetryOnExceptionAttribute` aspect: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionRetryAsync.cs#aspect)]

We can now apply the `[RetryOnException]` aspect to an async method: 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionRetryAsync.cs#async-method)]

Whenever the `SaveAsync` method is invoked, the `RetryOnExceptionAttribute.OnInvokeAsync` method is called instead. The aspect method will invoke the original method asynchronously and await its completion. If the asynchronous task throws an exception, the aspect will catch the exception and either retry the asynchronous call or rethrow the exception. 

When the database call fails twice before it succeeds, the program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionRetryAsync.txt)]


### Intercepting methods returning a null Task

The following example shows how to intercept methods that may return a null <xref:System.Threading.Tasks.Task>. There is no way for an aspect based on a semantically-advised <xref:PostSharp.Aspects.MethodInterceptionAspect> to return a null <xref:System.Threading.Tasks.Task>. If the aspect requires returning a null <xref:System.Threading.Tasks.Task>, it must use non-semantic advising. See <xref:semantic-advising> for details. 

[!code-csharp[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionNullTask.cs#example)]

When this aspect is applied to a method that returns `Task<string>`, and the method is called once with a non-null task and once with a null task, the program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Aspects/MethodInterceptionNullTask.txt)]

## See Also

**Reference**

<xref:PostSharp.Aspects.MethodInterceptionAspect>
<br><xref:PostSharp.Aspects.MethodInterceptionAspect.OnInvoke(PostSharp.Aspects.MethodInterceptionArgs)>
<br><xref:PostSharp.Aspects.MethodInterceptionArgs>
<br><xref:PostSharp.Aspects.MethodInterceptionArgs.Arguments>
<br><xref:PostSharp.Aspects.MethodInterceptionArgs.Binding>
<br><xref:PostSharp.Aspects.AdviceArgs.Instance>
<br><xref:PostSharp.Aspects.MethodInterceptionArgs.Method>
<br><xref:PostSharp.Serialization.PSerializableAttribute>

**Other Resources**

[PostSharp Aspect Framework - Product Page](https://www.postsharp.net/aspects)
