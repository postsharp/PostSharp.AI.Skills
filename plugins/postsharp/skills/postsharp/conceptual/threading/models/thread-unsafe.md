---
uid: thread-unsafe
redirect_from:
  - threading/thread-unsafe
title: "Thread-Unsafe Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains the thread-unsafe threading model in PostSharp, which throws exceptions when objects are accessed concurrently by multiple threads. It provides steps to apply this model to a class and details the rules enforced by this aspect."
---
# Thread-Unsafe Threading Model

When you are dealing with multithreaded code, you will run into situations where some objects are not safe for concurrent use by several threads. Although these objects should theoretically not be accessed concurrently, it is very hard to prove that it never happens. And when it does happen, thread-unsafe data structures get corrupted, and symptoms may appear much later. These issues are typically very difficult to debug. So instead of relying on hope, it would be nice if the object threw an exception whenever it is accessed simultaneously by several threads. This is why we have the thread-unsafe threading model.


## Applying the Thread-Unsafe model to a class


To apply the Thread-Unsafe threading model to a class:

1. Add the `PostSharp.Patterns.Threading` package to your project. 


2. Import the `PostSharp.Patterns.Threading` namespace into your file. 


3. Add the <xref:PostSharp.Patterns.Threading.ThreadUnsafeAttribute> to the class. 


4. Annotate your object model for parent/child relationships as described in <xref:aggregatable-adding>. 



## Rules enforced by the Thread-Unsafe aspect

The <xref:PostSharp.Patterns.Threading.ThreadUnsafeAttribute> aspect emits build-time errors in the following situations: 

* The class contains a public or internal field.

Internally, the Thread-Unsafe model is implemented by a lock. The lock is automatically acquired by public and internal methods, just like the Synchronized model.

A thread-unsafe object will throw the following exceptions:

* A <xref:PostSharp.Patterns.Threading.ThreadAccessException> whenever some code tries to access a field from a thread that does not own the correct lock, that is, the call stack does not contain a public or internal method of this object (for example, a private delegate call). 

* A <xref:PostSharp.Patterns.Threading.ConcurrentAccessException> when two public or internal methods execute at the same time on the same object (that is, whenever the lock cannot be acquired without waiting). 

These exceptions are thrown only when run-time verification is enabled, which is the default in the `Debug` build. See <xref:threading-runtime-verification>.

## See Also

**Reference**

<xref:PostSharp.Aspects.InstanceLevelAspect>
<br><xref:PostSharp.Patterns.Threading.ThreadUnsafeAttribute>
<br><xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute>
<br><xref:PostSharp.Patterns.Threading.EntryPointAttribute>

**Other Resources**

<xref:threading-waiving-verification>
<br><xref:ui-dispatching>
