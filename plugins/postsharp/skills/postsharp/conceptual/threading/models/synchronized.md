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

[!code-csharp[](../../../code/samples/Threading/SynchronizedLock.cs#example)]

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

[!code-csharp[](../../../code/samples/Threading/Synchronized.cs#service)]

To test this, we can run the following code.

[!code-csharp[](../../../code/samples/Threading/Synchronized.cs#program)]

The code above executes the `Process` method on two different threads: a background thread and the main thread. The `started` event makes sure that the background thread enters the method first. Because both threads access the same instance of `OrderService`, the background thread blocks the main thread until it exits the `Process` method. The `Thread.Sleep` call does not let the main thread in. When the background thread exits the method, the main thread is no longer blocked and continues its execution.

The output of this program is:

[!code-text[](../../../code/samples/Threading/Synchronized.txt)]


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
