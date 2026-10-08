---
uid: exception-handling
redirect_from:
  - custompatterns/aspects/tutorials/exception-handling
title: "Handling Exceptions"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp provides a solution to exception handling by encapsulating custom exception handling logic into a reusable class. This class can be applied as an attribute to methods and properties where exceptions are to be dealt with, making the process more efficient and code cleaner."
---
# Handling Exceptions

Adding exception handlers to code requires the addition of `try/catch` statements, which can quickly pollute code. Exception handling implemented this way is also not reusable, requiring the same logic to be implemented over and over wherever exceptions must be dealt with. Raw exceptions also present cryptic information and can often expose too much information to the user. 

PostSharp provides a solution to these problems by allowing custom exception handling logic to be encapsulated into a reusable class, which is then applied as an attribute to all methods and properties where exceptions are to be dealt with.


## Intercepting an exception

PostSharp provides the <xref:PostSharp.Aspects.OnExceptionAspect> class, which is the base class from which exception handlers are to be derived. 

If you also need to execute logic before and upon success of the target method, you can derive your aspect from the <xref:PostSharp.Aspects.OnMethodBoundaryAspect> class. Both classes behave almost identically as far as exception handling is concerned. 

Both classes define a virtual method named <xref:PostSharp.Aspects.OnExceptionAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)>. This is the method where the exception handling logic (that is, what would normally be in a `catch` statement) goes. A <xref:PostSharp.Aspects.MethodExecutionArgs> parameter is passed into this method by PostSharp; it contains information about the exception. 


### Creating an exception handling aspect

1. Add a reference to the *PostSharp* package to your project. 


2. Derive a class from <xref:PostSharp.Aspects.OnExceptionAspect>. 


3. Apply the <xref:PostSharp.Serialization.PSerializableAttribute> to the class. 


4. Override <xref:PostSharp.Aspects.OnExceptionAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> and implement your exception handling logic in this class. The <xref:System.Exception> object is available on the <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> property of the <xref:PostSharp.Aspects.MethodExecutionArgs> parameter. 


5. Add the aspect to one or more methods. Since <xref:PostSharp.Aspects.OnExceptionAspect> derives from the <xref:System.Attribute> class, you can add the aspect custom attribute to the methods you need. If you need to add the aspect to more methods (for instance all public methods in a namespace), you can learn about more advanced techniques in <xref:applying-aspects>. 



### Example

The following snippet shows an example of an exception handler which watches for exceptions of any type, and then writes a message to the console when an exception occurs:

```csharp
[PSerializable]
public class PrintExceptionAttribute : OnExceptionAspect
{

    public override void OnException(MethodExecutionArgs args)
    {
        Console.WriteLine(args.Exception.Message);
    }
}
```

Once created, apply the derived class to all methods and/or properties for which the exception handling logic is to be used, as shown in the following example:

```csharp
class Customer
{
    public string FirstName { get; set; }
    public string LastName { get; set; }

    [PrintException]
    public void StoreName(string path)
    {
        File.WriteAllText( path, string.Format( "{0} {1}", this.FirstName, this.LastName ) );
    }

}
```

Here `PrintException` will output a message when an exception occurs in trying to write text to a file. 


## Specifying the type of handled exceptions

The <xref:PostSharp.Aspects.OnExceptionAspect.GetExceptionType(System.Reflection.MethodBase)> method can be used to return the type of the exception which is to be handled by this aspect. Otherwise, all exceptions will be caught and handled by this class. Note that the <xref:PostSharp.Aspects.OnExceptionAspect.GetExceptionType(System.Reflection.MethodBase)> method is evaluated at build time. 

If the aspect needs to handle several types of exception, the `GetExceptionType` should return a common base type, and the `OnException` implementation should be modified to dynamically handle different types of exception. 


### Example

In the following snippet, we have updated the `PrintExceptionAttribute` aspect and added the possibility to specify, in the custom attribute constructor, which type of exception should be traced. 

```csharp
[PSerializable]
public class PrintExceptionAttribute : OnExceptionAspect
{
    Type type;

    public PrintExceptionAttribute(Type type)
    {
        this.type = type;
    }

    // Method invoked at build time.
    // Should return the type of exceptions to be handled. 
    public override Type GetExceptionType(MethodBase method)
    {
        return this.type;
    }


    public override void OnException(MethodExecutionArgs args)
    {
        Console.WriteLine(args.Exception.Message);
    }
}

class Customer
{
    public string FirstName { get; set; }
    public string LastName { get; set; }

    [PrintException(typeof(IOException))]
    public void StoreName(string path)
    {
        File.WriteAllText( path, string.Format( "{0} {1}", this.FirstName, this.LastName ) );
    }

}
```


## Ignoring ("swallowing") exceptions

The <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> member of <xref:PostSharp.Aspects.MethodExecutionArgs> in the exception handler's <xref:PostSharp.Aspects.OnExceptionAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> method can be set to ignore an exception. Note, however, that ignoring exceptions is generally dangerous and not recommended. In practice, it is only safe to ignore exceptions in event handlers (for example, to display a message in a WPF form) and in thread entry points. 

Exceptions can be ignored by setting the <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> property to `Return`. You must then set the return value of the method by setting the <xref:PostSharp.Aspects.MethodExecutionArgs.ReturnValue> property. 


### Example

The following aspect catches all exceptions flowing from the methods the aspect is applied to, prints the exception to the console, and makes the target method return `-1` instead of failing with an exception. 

```csharp
[PSerializable]
public class PrintAndIgnoreExceptionAttribute : OnExceptionAspect
{

    public override void OnException(MethodExecutionArgs args)
    {
        Console.WriteLine(args.Exception.Message);
        args.FlowBehavior = FlowBehavior.Return;
        args.ReturnValue = -1;
    }
}

public class Customer
{
    [PrintAndIgnoreException]
    public int GetDataLength(string path)
    {
       return File.ReadAllText(path).Length;
    }

}
```


## Replacing or wrapping exceptions

Often, the original exception must be hidden from the user or the client of the service, and should be replaced by another exception. This can be done by setting the <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior> property to `FlowBehavior.ThrowException` and the <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> to the new exception. 

* `FlowBehavior.RethrowException`: rethrows the original exception after the exception handler exits. This is the default behavior for the <xref:PostSharp.Aspects.OnExceptionAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> advice. 

* `FlowBehavior.ThrowException`: throws a new exception once the exception handler exits. This is useful when details of the original exception should be hidden from the user or when a more meaningful exception is to be shown instead. When throwing a new exception, a new exception object must be assigned to the `Exception` member of <xref:PostSharp.Aspects.MethodExecutionArgs>. The following snippet shows the creation of a new `BusinessExceptionAttribute` which throws a `BusinessException` containing a description of the cause: 

    ```csharp
    [PSerializable]
    public sealed class BusinessExceptionAttribute : OnExceptionAspect
    {
        public override void OnException(MethodExecutionArgs args)
        {
            Guid guid = Guid.NewGuid();
            args.FlowBehavior = FlowBehavior.ThrowException;        
            args.Exception = new BusinessException("Bad arguments. Use the id " + guid.ToString() + " for more information.");
        }
    }
    ```


> [!NOTE]
> You can also throw a new exception from the <xref:PostSharp.Aspects.OnExceptionAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> method, but the exception call stack will show the aspect method itself, while with <xref:PostSharp.Aspects.MethodExecutionArgs.FlowBehavior>, the exception call stack will originate from the target method (unless there is an interception aspect on the target method, in which case the call stack will originate from an intermediate method). 


### Example

The following aspect handles all exceptions in the `BusinessServices` class by generating a GUID for it, writing all details to the trace file and then throwing a `BusinessException` showing just the incident GUID and hiding other details. 

```csharp
[PSerializable]
public sealed class HandleExceptionsAttribute : OnExceptionAspect
{
    public override void OnException(MethodExecutionArgs args)
    {
        Guid guid = Guid.NewGuid();

        // In a real-world app, we would file the exception in the QA database.
        Trace.WriteLine( $"Exception {guid}:");
        Trace.WriteLine(args.Exception.ToString());

        args.FlowBehavior = FlowBehavior.ThrowException;
        args.Exception = new BusinessException( $"The service failed unexpectedly. Please report the incident to the QA team with the id #{guid}." );
    }
}              

[HandleExceptions]
public class BusinessServices
{ 
   // Dozens of methods here.
}

public class BusinessException : Exception
{
    public BusinessException(string message) : base(message)
    {            
    }
}
```


## Accessing the current execution context

The <xref:PostSharp.Aspects.OnExceptionAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)> method requires one argument of type <xref:PostSharp.Aspects.MethodExecutionArgs>. This object gives access to the exception being handled, the identity of the method being executed, its arguments, and the current object. 

The following table lists the pieces of context made available by the <xref:PostSharp.Aspects.MethodExecutionArgs> class. 

| Property | Description |
|----------|-------------|
| <xref:PostSharp.Aspects.MethodExecutionArgs.Method> | The method or constructor being executed (in case of generic methods, this property is set to the proper generic instance of the method). This is not necessarily equal to the method that originally threw the exception. |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Arguments> | The arguments passed to the method. In case of `out` and `ref` arguments, the argument values can be modified by the aspect.  |
| <xref:PostSharp.Aspects.AdviceArgs.Instance> | The object on which the method is being executed, that is, the value of the `this` keyword.  |
| <xref:PostSharp.Aspects.MethodExecutionArgs.Exception> | The <xref:System.Exception> thrown by the method. This value can be modified (see below).  |

> [!NOTE]
> The properties of the <xref:PostSharp.Aspects.MethodExecutionArgs> class cannot be directly viewed in the debugger. Because of optimizations, the properties must be referenced in your source code in order to be viewable in the debugger. 


### Example

The following code improves the previous example by adding more context information to the print-out of the exception details.

```csharp
[PSerializable]
public sealed class HandleExceptionsAttribute : OnExceptionAspect
{
    public override void OnException(MethodExecutionArgs args)
    {
        Guid guid = Guid.NewGuid();

        // In a real-world app, we would file the exception in the QA database.
        Trace.WriteLine( $"Exception {guid} when invoking the method {args.Method.DeclaringType.FullName}.{args.Method.Name}({string.Join(", ", args.Arguments)}) ");
        Trace.WriteLine(args.Exception.ToString());

        args.FlowBehavior = FlowBehavior.ThrowException;
        args.Exception = new BusinessException( $"The service failed unexpectedly. Please report the incident to the QA team with the id #{guid}." );
    }
}
```

## See Also

**Reference**

<xref:PostSharp.Aspects.OnExceptionAspect.OnException(PostSharp.Aspects.MethodExecutionArgs)>
<br><xref:PostSharp.Aspects.MethodExecutionArgs>
<br><xref:PostSharp.Aspects.OnExceptionAspect>
<br><xref:PostSharp.Serialization.PSerializableAttribute>
<br><xref:PostSharp.Aspects.OnExceptionAspect.GetExceptionType(System.Reflection.MethodBase)>

**Other Resources**

<xref:attribute-multicasting>
