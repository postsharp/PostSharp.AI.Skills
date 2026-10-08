---
uid: messages
redirect_from:
  - messaging/messages
title: "Working with Errors, Warnings, and Messages"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses how PostSharp, a compiler, can emit messages, warnings, and errors, and how custom code can use this feature. It also mentions an experimental feature in PostSharp 2.1."
---
# Working with Errors, Warnings, and Messages

Like any compiler, PostSharp can emit messages, warnings, and errors, which are commonly called *messages*. Custom code running at build time (typically the implementation of `CompileTimeValidate` or of a custom constraint) can use the PostSharp messaging facility to emit its own messages. 

In this section:

* <xref:ignoring-warnings>
* <xref:emitting-errors>


