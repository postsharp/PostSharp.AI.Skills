---
uid: manual-logging
title: "Adding Manual Logging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document guides on adding manual logging using PostSharp's LogAttribute aspect and LogSource class for custom error or warning messages, custom activities, and properties."
---
# Adding Manual Logging

When you use the <xref:PostSharp.Patterns.Diagnostics.LogAttribute> aspect, PostSharp automatically generates code that emits log records before and after the execution of a method. However, you may want to write your own records, for example to log a custom error or warning. Such messages should be displayed even when trace-level logging is disabled. When trace-level logging is enabled, they should appear in the right context and with the proper indentation. 

For these scenarios, you can use the methods provided by the <xref:PostSharp.Patterns.Diagnostics.LogSource> class. 


## In this chapter

| Section | Description |
|---------|-------------|
| <xref:log-custom-messages> | This article shows how to write standalone messages to the log. It discusses the difference between formatted-text messages and semantic messages, the latter being more appropriate for statistical processing. |
| <xref:log-custom-activities> | This article describes how to define custom activities, that is, nested scopes of operations that have a description and can succeed or fail. |
| <xref:log-properties> | This article shows how to add and configure properties to your custom messages and custom activities. |


