---
uid: backends
title: "Connecting to Source and Target Logging Frameworks"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions on how to configure PostSharp Logging to work with different logging frameworks and create custom logging backends. It also covers log collection and audit record addition."
---
# Connecting to Source and Target Logging Frameworks

This chapter explains how to configure PostSharp Logging to work with different logging frameworks.

PostSharp sends log events, both automatic and manual, to a *logging backend* that is responsible for outputting them on screen, into a file, database or elsewhere. PostSharp contains logging backends for many different logging frameworks and you can also create your own backend. 

These articles explain how to set up PostSharp with various logging frameworks:

* <xref:logging-console>
* <xref:logging-tracesource>
* <xref:logging-trace>
* <xref:nlog>
* <xref:log4net>
* <xref:serilog>
* <xref:etw>
* <xref:common-logging>
* <xref:application-insights>
* <xref:logging-aspnetcore>

In addition, the following scenarios are supported:

| Section | Description |
|---------|-------------|
| <xref:custom-logging-backend> | This article shows how to write PostSharp Logging output to any logging framework by building your own adapter, named *backend* in PostSharp terminology. |
| <xref:log-multiplexer> | This article explains how to write PostSharp Logging output to several logging frameworks, each possibly with different verbosity. |
| [Audit Sample](https://github.com/postsharp/PostSharp.Samples/tree/master/Diagnostics/PostSharp.Samples.Logging.Audit) | This example shows how to automatically add audit records to your application. |

PostSharp Logging can also collect events from many logging frameworks. This is called *collecting logs*. For details, see <xref:log-collecting>. 

## See Also

**Other Resources**

<xref:add-logging>

