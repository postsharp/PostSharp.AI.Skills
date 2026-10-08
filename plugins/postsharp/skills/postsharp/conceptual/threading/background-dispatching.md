---
uid: background-dispatching
title: "Dispatching a Method to Background"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp allows for the execution of a method in a background thread, reducing the need for boilerplate code. This is achieved by adding the PostSharp.Patterns.Threading package and the BackgroundAttribute to the desired method. "
---
# Dispatching a Method to Background

Long-running processes will block the further execution of code while the system waits for them to complete. When you are building applications it is common to push long-running processes to the background so that other processes can continue without waiting. Two common ways of doing this are with asynchronous processing and the <xref:System.ComponentModel.BackgroundWorker>. Both require a lot of boilerplate code to push execution to another thread. 

PostSharp provides you with the ability to push execution of a method to a background thread without having to worry about all of the boilerplate code.


## Executing a method in the background

1. Add the `PostSharp.Patterns.Threading` package to your project. 


2. Import the `PostSharp.Patterns.Threading` namespace into your file. 


3. Add the <xref:PostSharp.Patterns.Threading.BackgroundAttribute> to the method that you want to push to the background for execution. The return type of the method must be `void`, <xref:System.Threading.Tasks.Task> or <xref:System.Threading.Tasks.Task`1>. 


These steps are all that is required for you to declare that a method should be executed in a background thread.

If the method is `void`, it will be executed in the background, and the caller code will not wait until the background method completes its execution. If the method returns a <xref:System.Threading.Tasks.Task>, the method will be fully executed in the background (even the first segment of the method, before the first `await` keyword). 

Methods annotated with this attribute are run in the managed thread pool, unless you use the attribute's <xref:PostSharp.Patterns.Threading.BackgroundAttribute.IsLongRunning> property, in which case it is run as a new separate background thread. 

## See Also

**Reference**

<xref:System.ComponentModel.BackgroundWorker>

**Other Resources**

<xref:ui-dispatching>
