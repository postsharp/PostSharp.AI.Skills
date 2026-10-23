---
uid: thread-affine
redirect_from:
  - threading/thread-affine
title: "Thread Affine Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains the Thread Affine threading model in PostSharp, which restricts object instance access to the thread that created it. It illustrates how to apply this model to a class and its enforcement rules."
---
# Thread Affine Threading Model

One of the simplest ways to consider threading is to limit object instance access to the thread that created the instance. This is how the Thread Affine threading model works.


## Adding the Thread Affine model to a class


To apply the Thread-Affine threading model to a class:

1. Add the `PostSharp.Patterns.Threading` package to your project. 


2. Import the `PostSharp.Patterns.Threading` namespace into your file. 


3. Add the <xref:PostSharp.Patterns.Threading.ThreadAffineAttribute> to the class. 


4. Annotate your object model for parent/child relationships as described in <xref:aggregatable-adding>. 



### Example

In the example below, the <xref:PostSharp.Patterns.Threading.ThreadAffineAttribute> has been added to the class. 

[!code-csharp[](../../../code/samples/Threading/Core/ThreadAffine.cs#service)]


## Rules enforced by the Thread-Affine aspect



Except for the types of fields, the <xref:PostSharp.Patterns.Threading.ThreadAffineAttribute> aspect does not verify your code at build time. Unlike other threading models, it allows public and internal fields. Instead, it injects code that enforces the model at run time. If it detects that the object is being accessed from a different thread than the one that created it, the aspect will throw a <xref:PostSharp.Patterns.Threading.ThreadMismatchException> exception. This verification happens only when run-time verification is enabled, which is the default in the `Debug` build. See <xref:threading-runtime-verification>. 

To test the thread-affine `OrderService` class, we can run the following code: 

[!code-csharp[](../../../code/samples/Threading/Core/ThreadAffine.cs#program)]

The output of this program is:

[!code-text[](../../../code/samples/Threading/Core/ThreadAffine.txt)]

The `orderService.Process(1)` call succeeds because the `orderService` instance is created (with the `new` keyword) and used on the same thread.

Then, a background thread executes the `orderService.Process(2)` method, and the program writes the exception to the console. A <xref:PostSharp.Patterns.Threading.ThreadMismatchException> exception is thrown because the `orderService` instance was created on the main thread and the background thread attempted to use it. Because the `OrderService` class has been marked with the <xref:PostSharp.Patterns.Threading.ThreadAffineAttribute> attribute, only the thread that creates an instance of it can access that instance. 


## Working with object trees

Because the Thread-Affine model is an implementation of the Aggregatable pattern, all of the same behaviors of the <xref:PostSharp.Patterns.Model.AggregatableAttribute> are available. For more information regarding object trees, see <xref:aggregatable>. 

> [!NOTE]
> Once you have established your parent-child relationships, you will need to apply compatible threading models to the child classes. You will want to refer to the <xref:threading-model-compatibility> article to determine which threading model will work for the children of the Thread-Affine object. 

## See Also

**Other Resources**

<xref:threading-model-compatibility>

**Reference**

<xref:PostSharp.Patterns.Threading.ThreadAffineAttribute>
<br><xref:PostSharp.Patterns.Threading.ThreadMismatchException>
<br><xref:PostSharp.Patterns.Threading.PrivateThreadAwareAttribute>
<br><xref:PostSharp.Patterns.Model.ChildAttribute>
