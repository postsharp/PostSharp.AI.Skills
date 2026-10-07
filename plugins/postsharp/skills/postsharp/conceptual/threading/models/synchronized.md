---
uid: synchronized
redirect_from:
  - threading/synchronized
title: "Synchronized Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains the Synchronized threading model in C#. It details how to apply the model, its differences from the 'lock(this)' statement, and how it works with object trees."
---
# Synchronized Threading Model

A common way to avoid data races is to enclose all public instance methods of a class with a `lock(this)` statement. This is basically what the Synchronized model does.

This article describes how to use the Synchronized model and how it differs from the `lock(this)` statement. 


## Comparing with lock(this)

Traditionally, the C# `lock(this)` statement has been used to synchronize access of several threads to a single object. When an object is locked by one thread, any other thread that attempts to access that object will have its execution blocked. 

```csharp
private object myLockingObject = new Object();
        
public void DoSomething()
{
    lock(myLockingObject)
    {
        //some code that does something in one thread at a time
    }
}
```

In this example, the `myLockingObject` member variable is used as a locking object. Once a thread runs the `lock(myLockingObject)` line, all other threads that enter the `DoSomething` method will stop executing, or be blocked, until the original thread has exited the `lock(myLockingObject)` code block. 

The Synchronized model is similar to using the `lock` statement around every single public method, but it has the following differences: 

* It is not technically equivalent to locking the current instance (`this`). Another object is actually being locked. 

* Locking is automatic for all public and internal instance methods. You cannot forget it.

* If a thread attempts to access a field without having first acquired access to the object (by invoking a public or internal method), an exception will be thrown.

* The pattern also works with entities composed of several objects organized in a tree.


## Applying the Synchronized model to a class


To apply the Synchronized threading model to a class:

1. Add the `PostSharp.Patterns.Threading` package to your project. 


2. Import the `PostSharp.Patterns.Threading` namespace into your file. 


3. Add the <xref:PostSharp.Patterns.Threading.SynchronizedAttribute> to the class. 


4. Annotate your object model for parent/child relationships as described in <xref:aggregatable-adding>. 



### Example

In the example below, the <xref:PostSharp.Patterns.Threading.SynchronizedAttribute> has been added to the class. 

```csharp
[Synchronized]
public class OrderService
{
    public void Process(int sequence)
    {
        Console.WriteLine("sequence {0}", sequence);
        Console.WriteLine("sleeping for 10s");
        
        Thread.Sleep(new TimeSpan(0,0,10));
    }
}
```

To test this, we can run the following code.

```csharp
public void Main()
{
    var orderService = new OrderService();
    
    var backgroundWorker = new BackgroundWorker();
    backgroundWorker.DoWork += (sender, args) => orderService.Process(1);
    backgroundWorker.RunWorkerAsync();
    
    orderService.Process(2);
}
```

The code above will attempt to execute the `Process` method on two different threads: the main thread and a background worker thread. Because these two threads are trying to access the same instance of the `OrderService` the first thread to access it will block the second. As a result, when you run the program you will first see the following. 

![](Synchronized5.PNG)

Because the `OrderService.Process` method has a `Thread.Sleep` call, the first thread accessing that method will block the second for 10 seconds. After those 10 seconds have passed the second thread will no longer be blocked and it will be able to continue its execution. 

![](Synchronized6.PNG)


## Rules enforced by the Synchronized aspect

The <xref:PostSharp.Patterns.Threading.SynchronizedAttribute> aspect emits build-time errors in the following situations: 

* The class contains a public or internal field.

A synchronized object will throw a <xref:PostSharp.Patterns.Threading.ThreadAccessException> whenever some code tries to access a field from a thread that does not own the correct lock, that is, the call stack does not contain a public or internal method of this object (for example, a private delegate call). 


## Working with object trees

Because the Synchronized model is an implementation of the Aggregatable pattern, all of the same behaviors of the <xref:PostSharp.Patterns.Model.AggregatableAttribute> are available. For more information regarding object trees, see <xref:aggregatable>. 

> [!NOTE]
> Once you have established your parent-child relationships, you will need to apply compatible threading models to the child classes. You will want to refer to the <xref:threading-model-compatibility> article to determine which threading model will work for the children of the Synchronized object. 

## See Also

**Other Resources**

<xref:threading-model-compatibility>

**Reference**

<xref:PostSharp.Patterns.Threading.SynchronizedAttribute>
<br><xref:PostSharp.Patterns.Model.ChildAttribute>
<br><xref:PostSharp.Patterns.Threading.PrivateThreadAwareAttribute>
