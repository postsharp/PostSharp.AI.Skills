---
uid: logging
title: "Logging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp Logging is a tool that allows detailed, customizable logging for applications. It supports various logging frameworks and structured log servers, and is especially useful for distributed systems."
---
# Logging

PostSharp Logging allows you to add logging to your complete solution in just a few lines of code.

It has the following benefits:

* Can produce very detailed logs including parameter values and execution time.

* Can stay turned off by default and be dynamically enabled on demand, with fine-grained control over types and namespaces, when a problem happens in production. This can be done just by editing a configuration file in an online file hosting service such as Google Drive.

* Fast, and often faster than `string.Format`. 

* Fully customizable. Works with your logging framework (even your custom one).

* Supports a broad set of logging frameworks.

* Supports structured log servers (such as Elasticsearch or Application Insights) and statistical processing of messages.

* Supports distributed systems and helps generate correlation information.


## In this chapter

| Section | Description |
|---------|-------------|
| <xref:add-logging> | This article shows how to add detailed logging to your application. |
| <xref:backends> | This article shows how to connect PostSharp Logging to supported logging frameworks. |
| <xref:logging-customizing> | This article describes how to customize the log records. It covers both build-time configuration (logging profiles) and run-time configuration. |
| <xref:manual-logging> | This article explains how to add manual log records and custom activities to your log. |
| <xref:log-enabling> | This article shows how to enable and disable logging for a specific type and namespace, and message severity. |
| <xref:distributed-logging> | This article explains how to configure PostSharp Logging to make sure that the produced logs are useful and meaningful in a distributed system. |
| <xref:custom-formatter> | This article shows how to create your own formatter to display parameter values. |
| <xref:custom-logging-backend> | This article demonstrates how to use your own logging framework with PostSharp Logging. |
| <xref:logging-exception-handling> | This article explains what happens when the logging component fails and how to customize this behavior. |

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.LogAttribute>
<br><xref:PostSharp.Patterns.Diagnostics.LogExceptionAttribute>

