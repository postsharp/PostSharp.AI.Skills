---
uid: threading-models
redirect_from:
  - threading/threading-models
title: "Writing Thread-Safe Code with Threading Models"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses the concept of threading models, their benefits in multithreaded programming, and the various threading models provided by the PostSharp Threading Library. It also provides links to related topics and resources."
---
# Writing Thread-Safe Code with Threading Models

A threading model is a design pattern that gives guarantees that your code executes safely on a multithreaded computer. Threading models both define coding rules (for instance: all fields must be private) and add new behaviors to existing code (for instance: acquiring a lock before method execution). Coding rules are typically enforced at build time or at run time; violations result in build-time errors or run-time exceptions. Threading models may also require the use of custom attributes in source code, for instance to indicate that a method requires read access to the object.

> [!TIP]
> We recommend assigning a threading model to every class whose instances can be shared between different threads.

Threading models raise the level of abstraction at which multithreading is addressed. Compared to working directly with locks and other low-level threading primitives, using threading models has the following benefits:

* Threading models are named solutions to a recurring problem. Threading models are specific types of design patterns, and have the same benefits. When team members discuss the multithreaded behavior of a class, they need to know only which threading model this class uses. They do not need to know the details of its implementation. Since the human short-term memory seems to be limited to 5-9 elements, it is important to think in terms of larger conceptual blocks whenever we can. 

* Much of the code required to implement the threading model can be automatically generated, which decreases the number of lines of code, and therefore the number of defects. It also reduces development and maintenance costs. 

* Your source code can be automatically verified against the selected threading model, both at build time and at run time. This makes the discovery of defects much more deterministic. Without verifications, threading defects usually show up randomly and provoke data structure corruption instead of immediate exceptions. Run-time verification would be too labor-intensive to implement without compiler support, so it would most likely be omitted. 


## Available threading models

PostSharp Threading Library provides an implementation for the following threading models:

| Threading model | Aspect type | Description |
|-----------------|-------------|-------------|
| <xref:thread-unsafe> | <xref:PostSharp.Patterns.Threading.ThreadUnsafeAttribute> | These objects may never be accessed concurrently by several threads. |
| <xref:thread-affine> | <xref:PostSharp.Patterns.Threading.ThreadAffineAttribute> | These objects must be accessed from the thread that instantiated them. |
| <xref:synchronized> | <xref:PostSharp.Patterns.Threading.SynchronizedAttribute> | Synchronized objects can be accessed by a single thread at a time. Other threads will wait until the object is available. |
| <xref:reader-writer-synchronized> | <xref:PostSharp.Patterns.Threading.ReaderWriterSynchronizedAttribute> | These objects can be read concurrently by several threads, but write access requires exclusivity. Public methods of this object must specify which kind of access they require (read or write, typically). |
| <xref:actor> | <xref:PostSharp.Patterns.Threading.ActorAttribute> | These objects communicate with their clients using an asynchronous communication pattern. All accesses to the object are queued and then processed in a single thread. However, queuing is transparent to clients, which call standard `void` or `async` methods.  |
| <xref:freezable> | <xref:PostSharp.Patterns.Threading.FreezableAttribute> | These objects can be set to a state where their property values can no longer be changed. Unlike immutable objects, the developer dictates the time and place in their code where changes to the object's state will no longer be accepted. |
| <xref:immutable> | <xref:PostSharp.Patterns.Threading.ImmutableAttribute> | These objects cannot have their state changed after their constructor has finished executing. |


## Other topics

| Article | Description |
|---------|-------------|
| <xref:thread-safety-policy> | This article describes how to get compiler warnings when you forget to assign a threading model to a type. |
| <xref:threading-waiving-verification> | This article shows how to disable the enforcement of the threading model for specific fields or methods. |
| <xref:threading-model-compatibility> | This article lists compatibility of threading models when they are applied to objects that are in a parent-child relationship. |
| <xref:threading-runtime-verification> | This article explains when run-time verification is enabled or disabled and how to customize the default behavior. |


## Conceptual documentation

Read [this technical white paper](https://www.postsharp.net/links/threading-model-white-paper) for details about the concepts and architecture of PostSharp Threading Models. 


