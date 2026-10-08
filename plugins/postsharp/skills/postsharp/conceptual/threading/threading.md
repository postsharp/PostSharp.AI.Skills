---
uid: threading-library
redirect_from:
  - threading/threading-library
title: "Multithreading"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The Threading Pattern Library aids in building multithreaded applications with fewer lines of code and defects, by implementing locking models and thread synchronization aspects."
---
# Multithreading

The Threading Pattern Library helps you build multithreaded applications with fewer lines of code and fewer defects. The library implements threading models (for instance thread unsafe, reader-writer synchronized, actor) and thread dispatching aspects (to background thread, to UI thread).

It provides the following features:

* **Threading Models.** A threading model is a design pattern that gives guarantees that your code executes safely on a multithreaded computer. Seven models are implemented: Thread Unsafe, Thread Affine, Synchronized, Reader-Writer Synchronized, Actor, Freezable and Immutable. See <xref:threading-models> for details. 

* **Thread Dispatching.** Custom attributes <xref:PostSharp.Patterns.Threading.DispatchedAttribute> and <xref:PostSharp.Patterns.Threading.BackgroundAttribute> cause the execution of a method to be dispatched to the UI thread or to a background thread, respectively. For details, see <xref:ui-dispatching> and <xref:background-dispatching>. 

