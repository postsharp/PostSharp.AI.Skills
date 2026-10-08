---
uid: method-decorator
redirect_from:
  - custompatterns/aspects/tutorials/method-decorator
title: "Injecting Behaviors Before and After Method Execution"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides an in-depth guide on how to use the PostSharp.Aspects.OnMethodBoundaryAspect aspect in PostSharp for injecting behaviors before and after method execution. It covers various scenarios including handling exceptions, sharing state between advices, working with async and iterator methods."
---
# Injecting Behaviors Before and After Method Execution

The <xref:PostSharp.Aspects.OnMethodBoundaryAspect> aspect implements the so-called *decorator* pattern: it allows you to execute logic before and after the execution of a target method. 

You may want to use method decorators to perform logging, monitor performance, initialize database transactions or any one of many other infrastructure-related tasks. PostSharp provides you with a framework for all of these tasks in the form of the <xref:PostSharp.Aspects.OnMethodBoundaryAspect>. 


## Executing code before and after method execution

When you are decorating methods, there are different locations that you may wish to inject functionality to. You may want to perform a task prior to the method executing or just before it finishes execution. There are situations where you may want to inject functionality only when the method has successfully executed or when it has thrown an exception. All of these injection points are structured and available to you in the <xref:PostSharp.Aspects.OnMethodBoundaryAspect> class as virtual methods (called *advices*) that you can implement if you need them. 

The following table shows the advice methods available in the <xref:PostSharp.Aspects.OnMethodBoundaryAspect> class (see below for more advice methods available on `async` and iterator methods). 

| Advice | Description |
|--------|-------------|
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution starts, before any user code. |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnSuccess(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution succeeds (that is, returns without an exception), after any user code. |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution fails with an exception, after any user code. It is equivalent to a `catch` block.  |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnExit(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution exits, whether successfully or with an exception. This advice runs after any user code and after the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnSuccess(PostSharp.Aspects.MethodExecutionArgs)> or <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> method of the current aspect. It is equivalent to a `finally` block.  |


### Creating a simple aspect that writes text when a method enters, succeeds, or fails

1. Add a reference to the *PostSharp* package to your project. 


2. Create an aspect class and derive it from <xref:PostSharp.Aspects.OnMethodBoundaryAspect>. 


3. Annotate the class with the [<xref:PostSharp.Serialization.PSerializableAttribute>] custom attribute. 


4. Override the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)>, <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnSuccess(PostSharp.Aspects.MethodExecutionArgs)>, <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> or <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnExit(PostSharp.Aspects.MethodExecutionArgs)> as needed, and code the logic that needs to be executed at these points. 


5. Add the aspect to one or more methods. Since <xref:PostSharp.Aspects.OnMethodBoundaryAspect> derives from the <xref:System.Attribute> class, you can add the aspect custom attribute to the methods you need. If you need to add the aspect to more methods (for instance all public methods in a namespace), you can learn about more advanced techniques in <xref:applying-aspects>. 



### Example

The following code snippet shows a simple aspect based on <xref:PostSharp.Aspects.OnMethodBoundaryAspect> which writes a line to the console during each of the four events. The aspect is applied to the `Program.Main` method. 

```csharp
[PSerializable]
public class LoggingAspect : OnMethodBoundaryAspect
{

  public override void OnEntry(MethodExecutionArgs args)
  {
     Console.WriteLine("The {0} method has been entered.", args.Method.Name);
  }
  
  public override void OnSuccess(MethodExecutionArgs args)
  {
      Console.WriteLine("The {0} method executed successfully.", args.Method.Name);
  }
  
  public override void OnExit(MethodExecutionArgs args)
  {
     Console.WriteLine("The {0} method has exited.", args.Method.Name);
  }     
 
  public override void OnException(MethodExecutionArgs args)
  {
      Console.WriteLine("An exception was thrown in {0}.", args.Method.Name);
  }

}

static class Program
{
   [LoggingAspect]
   static void Main()
   {
     Console.WriteLine("Hello, world.");
   }
}
```

Executing the program prints the following lines to the console:

```none
The Main method has been entered.
Hello, world.
The Main method executed successfully.
The Main method has exited.
```


## Accessing the current execution context

As illustrated in the example above, you can access information about the method being intercepted from the property <xref:PostSharp.Aspects.MethodExecutionArgs.Method>, which gives you a reflection object <xref:System.Reflection.MethodBase>. This object gives you access to parameters, return type, declaring type, and other characteristics. In case of generic methods or generic types, <xref:PostSharp.Aspects.MethodExecutionArgs.Method> gives you the proper generic method instance, so you can use this object to get generic parameters. 

The <xref:PostSharp.Aspects.MethodExecutionArgs> object contains more information about the current execution context, as illustrated in the following table: 

| Property | Description |
|----------|-------------|
| <xref:PostSharp.Aspects.MethodExecutionArgs.Method> | The method or constructor being executed (in case of generic methods, this property is set to the proper generic instance of the method). |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Arguments> | The arguments passed to the method. In case of `out` and `ref` arguments, the argument values can be modified by the implementation of the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnSuccess(PostSharp.Aspects.MethodExecutionArgs)> or <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnExit(PostSharp.Aspects.MethodExecutionArgs)> advices.  |
| <xref:PostSharp.Aspects.AdviceArgs.Instance> | The object on which the method is being executed, that is, the value of the `this` keyword.  |
| <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue> | The return value of the method. This property can be modified by the aspect. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> | The <xref:System.Exception> thrown by the method. This value can be modified (see below).  |

> [!NOTE]
> The properties of the <xref:PostSharp.Aspects.MethodExecutionArgs> class cannot be directly viewed in the debugger. Because of optimizations, the properties must be referenced in your source code in order to be viewable in the debugger. 


### Example

The following program illustrates how to consume the current context from the <xref:PostSharp.Aspects.MethodExecutionArgs> parameter: 

```csharp
[PSerializable]
public class LoggingAspect : OnMethodBoundaryAspect
{

  public override void OnEntry(MethodExecutionArgs args)
  {
     Console.WriteLine("Method {0}({1}) started.", args.Method.Name, string.Join( ", ", args.Arguments ) );
  }
  
  public override void OnSuccess(MethodExecutionArgs args)
  {
      Console.WriteLine("Method {0}({1}) returned {2}.", args.Method.Name, string.Join( ", ", args.Arguments ), args.ReturnValue );
  }
 

}

static class Program
{
   static void Main()
   {
     Foo( 1, 2 );
   }
   
   [LoggingAspect]
   static int Foo(int a, int b)
   {
       Console.WriteLine("Hello, world.");
       return 3;
   }
}
```

When this program executes, it prints the following output:

```none
Method Foo(1, 2) started.
Hello, world.
Method Foo(1, 2) returned 3.
```


## Returning without executing the method

When your aspect is interacting with the target code, there are situations where you will need to alter the execution flow behavior. For example, your <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)> advice may want to prevent the target method from being executed. PostSharp offers this ability through the use of the <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> property. Unless the target method is `void` or is an iterator method, you will also need to set the <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue> property. 

```csharp
public override void OnEntry(MethodExecutionArgs args)
{
    if (args.Arguments.Count > 0 && args.Arguments[0] == null)
    {
        args.FlowBehavior = FlowBehavior.Return;
        args.ReturnValue = -1;
    }

    Console.WriteLine("The {0} method was entered with the parameter values: {1}",
                      args.Method.Name, string.Join(", ", args.Arguments));
}
```

To exit the execution of the target code, set the <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> property on the <xref:PostSharp.Aspects.MethodExecutionArgs> to `Return`. 

When you manage execution flow control with exceptions, there are two primary situations that you need to consider: rethrowing the exception and throwing a new exception.


## Handling exceptions

When you implement the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> method, PostSharp will generate a `try/catch` block and invoke <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> from the `catch` block. 

The default behavior of the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> advice is to rethrow the exception after the execution of the advice. You can also choose to ignore the exception or to replace it with another one. For details, see <xref:exception-handling>. 


## Sharing state between advices

When you are working with multiple advices on a single aspect, you will encounter the need to share state between these advices. For example, if you have created an aspect that times the execution of a method, you will need to track the starting time at <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)> and share that with <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnExit(PostSharp.Aspects.MethodExecutionArgs)> to calculate the duration of the call. 

To do this, use the <xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag> property on the <xref:PostSharp.Aspects.MethodExecutionArgs> parameter in each of the advices. Because <xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag> is an object type, you will need to cast the value stored in it while retrieving it and before using it. 

```csharp
[PSerializable]
public class ProfilingAspect : OnMethodBoundaryAspect
{
    public override void OnEntry(MethodExecutionArgs args)
    {
        args.MethodExecutionTag = Stopwatch.StartNew();
    }

    public override void OnExit(MethodExecutionArgs args)
    {
        var sw = (Stopwatch)args.MethodExecutionTag;
        sw.Stop();

        System.Diagnostics.Debug.WriteLine("{0} executed in {1} seconds", args.Method.Name,
                                           sw.ElapsedMilliseconds / 1000);
    }
}
```

> [!NOTE]
> The value stored in <xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag> will not be shared between different instances of the aspect. If the aspect is attached to two different pieces of target code, each attachment will have its own unshared <xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag> for state storage. 


## Working with async methods

A specific characteristic of `async` methods is that their execution can be suspended while they are awaiting a dependent operation (typically another `async` method or a <xref:System.Threading.Tasks.Task>). While an `async` method is suspended, it does not block any thread. When the dependent operation has completed, the execution of the `async` method can be resumed, possibly on a different thread than the one the method was previously executing on. 

There are many situations in which you may want to execute some logic when an `async` method is being suspended or resumed. For instance, a profiling aspect may exclude the time when the method is waiting for a dependency. You can achieve this by overriding the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnYield(PostSharp.Aspects.MethodExecutionArgs)> and <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnResume(PostSharp.Aspects.MethodExecutionArgs)> methods of the <xref:PostSharp.Aspects.OnMethodBoundaryAspect> class. 

The following table shows the advices that are specific to `async` methods (and iterator methods). 

| Advice | Description |
|--------|-------------|
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnYield(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution is suspended, that is, when the operand of the `await` operator is a task that has not yet completed.  |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnResume(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution is resumed, that is, when the operand of the `await` operator has completed.  |

> [!NOTE]
> When the operand of the `await` is a task that has already completed, the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnYield(PostSharp.Aspects.MethodExecutionArgs)> and <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnResume(PostSharp.Aspects.MethodExecutionArgs)> methods are not invoked. 

See <xref:semantic-advising> for more details regarding the behavior <xref:PostSharp.Aspects.OnMethodBoundaryAspect> aspect on asynchronous methods. 


### Example

In this example, we will reuse the `ProfilingAspect` class from the previous section and will extend it to exclude the time spent while waiting for dependent operations. 

```csharp
[PSerializable]
public class ProfilingAspect : OnMethodBoundaryAspect
{
    public override void OnEntry(MethodExecutionArgs args)
    {
      args.MethodExecutionTag = Stopwatch.StartNew();
    }

    public override void OnExit(MethodExecutionArgs args)
    {
      var sw = (Stopwatch)args.MethodExecutionTag;
      sw.Stop();

      Console.WriteLine("Method {0} executed for {1}ms.", args.Method.Name,
                        sw.ElapsedMilliseconds);
    }
    
    public override void OnYield( MethodExecutionArgs args )
    {
      Stopwatch sw = (Stopwatch) args.MethodExecutionTag;
      sw.Stop();
    }
    
    public override void OnResume( MethodExecutionArgs args )
    {
        Stopwatch sw = (Stopwatch) args.MethodExecutionTag;
        sw.Start();
    }
    
}
```

Apply the `[ProfilingAspect]` attribute to the `TestProfiling` method. 

```csharp
[ProfilingAspect]
public async Task TestProfiling()
{
    await Task.Delay( 3000 );
    Thread.Sleep( 1000 );
}
```

During the code execution, the stopwatch will start upon entering the `TestProfiling` method. It will stop before the `await` statement and resume when the awaited task is done. Finally, the time measurement is stopped again before exiting the `TestProfiling` method and the result is written to the console. 

```none
Method TestProfiling executed for 1007ms.
```


## Working with iterator methods

Iterator methods are methods that contain the `yield` keyword. Internally, the C# or VB compiler transforms the iterator method into a state machine class that implements the <xref:System.Collections.Generic.IEnumerable`1> and <xref:System.Collections.Generic.IEnumerator`1> interfaces. Calling the <xref:System.Collections.IEnumerator.MoveNext> method causes the method to execute until the next `yield` keyword. The keyword causes the method execution to be suspended, and it is resumed by the next call to <xref:System.Collections.IEnumerator.MoveNext>. 

Just like with `async` methods, you can use the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnYield(PostSharp.Aspects.MethodExecutionArgs)> and <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnResume(PostSharp.Aspects.MethodExecutionArgs)> methods to inject behaviors when an iterator method is suspended or resumed. 

The following table explains the behavior of the different advices of the <xref:PostSharp.Aspects.OnMethodBoundaryAspect> aspects in the context of iterator methods. 

| Advice | Description |
|--------|-------------|
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution starts, that is, during the first call of the <xref:System.Collections.IEnumerator.MoveNext> method, before any user code.  |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnYield(PostSharp.Aspects.MethodExecutionArgs)> | When the iterator method emits a result using the `yield return` statement (and the iterator execution is therefore suspended). The operand of the `yield return` statement (that is, the value of the <xref:System.Collections.IEnumerator.Current> property) can be read from the <xref:PostSharp.Aspects.MethodExecutionArgs.YieldValue> property of the <xref:PostSharp.Aspects.MethodExecutionArgs> parameter.  |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnResume(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution is resumed by a call to <xref:System.Collections.IEnumerator.MoveNext>. Note, however, that the first call to <xref:System.Collections.IEnumerator.MoveNext> results in a call to <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)>.  |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnSuccess(PostSharp.Aspects.MethodExecutionArgs)> | When the iterator method execution succeeds (that is, returns without an exception, causing the <xref:System.Collections.IEnumerator.MoveNext> method to return `false`) or is interrupted (by disposing of the enumerator), after any user code.  |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> | When the method execution fails with an exception, after any user code. It is equivalent to a `catch` block around the whole iterator method.  |
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnExit(PostSharp.Aspects.MethodExecutionArgs)> | When the iterator method execution succeeds (when <xref:System.Collections.IEnumerator.MoveNext> method returns `false`), is interrupted (by disposing of the enumerator) or fails with an exception. This advice runs after any user code and after the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnSuccess(PostSharp.Aspects.MethodExecutionArgs)> or <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> method of the current aspect. It is equivalent to a `finally` block around the whole iterator.  |

See <xref:semantic-advising> for more details regarding the behavior <xref:PostSharp.Aspects.OnMethodBoundaryAspect> aspect on iterator methods. 


### Example

The following program illustrates the timing of different advices in the context of an iterator.

```csharp
[PSerializable]
    class MyAspect : OnMethodBoundaryAspect
    {
        public override void OnEntry(MethodExecutionArgs args)
        {
            Console.WriteLine("! entry");
        }

        public override void OnResume(MethodExecutionArgs args)
        {
            Console.WriteLine("! resume");
        }
        public override void OnYield(MethodExecutionArgs args)
        {
            Console.WriteLine($"! yield return {args.YieldValue}");
        }

        public override void OnSuccess(MethodExecutionArgs args)
        {
            Console.WriteLine("! success");
        }

        public override void OnExit(MethodExecutionArgs args)
        {
            Console.WriteLine("! exit");
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            foreach (var i in Foo())
            {
                Console.WriteLine($"# received {i}");
            }

            Console.WriteLine("# done");
        }

        [MyAspect]
        static IEnumerable<int> Foo()
        {
            Console.WriteLine("@ part 1");
            yield return 1;
            Console.WriteLine("@ part 2");
            yield return 2;
            Console.WriteLine("@ part 3");
            yield return 3;
            Console.WriteLine("@ part 4");
        }
    }
```

Executing the program prints the following output:

```none
! entry
@ part 1
! yield return 1
# received 1
! resume
@ part 2
! yield return 2
# received 2
! resume
@ part 3
! yield return 3
# received 3
! resume
@ part 4
! success
! exit
# done
```


### Working with other enumerable methods

The behavior described above is also true when there is a method interception aspect ordered after the method boundary aspect. In that case, the method interception aspect could return any enumerable object, not just the iterator from the target iterator method. This means that the `OnYield` advice behaves differently: 

| Advice | Description |
|--------|-------------|
| <xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnYield(PostSharp.Aspects.MethodExecutionArgs)> | At the end of the returned enumerator's <xref:System.Collections.IEnumerator.MoveNext> method  |

You can also enable this behavior for other methods with the return type <xref:System.Collections.Generic.IEnumerable`1> or <xref:System.Collections.Generic.IEnumerator`1> using semantic advising. See <xref:semantic-advising> for more details. 


### Technical details

When there is a non-semantically-advised method boundary aspect or a method interception aspect ordered after a semantically-advised method boundary aspect on an iterator method, or when a semantically-advised method boundary aspect is applied to a method that returns <xref:System.Collections.Generic.IEnumerable`1>, <xref:System.Collections.Generic.IEnumerator`1> or their non-generic variants, PostSharp transforms the target method equivalently to this example: 

Suppose the original target method is:

```csharp
[MySemanticBoundaryAspect, MyInterceptionAspect] // in that order
static IEnumerable<int> Return2()
{
	yield return 2;
}
```

Then that piece of code is equivalent to this code:

```csharp
[MySemanticBoundaryAspect]
static IEnumerable<int> Return2()
{
    var innerEnumerable = Return2__OriginalMethod();
    var innerEnumerator = innerEnumerable?.GetEnumerator();
    if (innerEnumerator == null) 
    {
        yield break;
    }
    foreach(var element in innerEnumerator)
    {
        yield return element;
    }
}
[MyInterceptionAspect]
static IEnumerable<int> Return2__OriginalMethod()
{
	yield return 2;
}
```

## See Also

**Reference**

<xref:PostSharp.Aspects.OnMethodBoundaryAspect>
<br><xref:PostSharp.Aspects.Arguments>
<br><xref:PostSharp.Aspects.MethodExecutionArgs.MethodExecutionTag>
<br><xref:PostSharp.Serialization.PSerializableAttribute>
<br><xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnExit(PostSharp.Aspects.MethodExecutionArgs)>
<br><xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnSuccess(PostSharp.Aspects.MethodExecutionArgs)>
<br><xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)>
<br><xref:PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)>
<br><xref:PostSharp.Aspects.MethodExecutionArgs.Method>
<br><xref:PostSharp.Aspects.MethodExecutionArgs.Arguments>

**Other Resources**

[PostSharp Aspect Framework - Product Page](https://www.postsharp.net/aspects)
