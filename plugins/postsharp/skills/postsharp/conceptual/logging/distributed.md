---
uid: distributed-logging
redirect_from:
  - logging/distributed-logging
title: "Logging in a Distributed System"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses the challenges and solutions of implementing logging in a distributed system using PostSharp Logging. It explains how to prepare applications for distributed logging, use hierarchical context IDs, and compare generation strategies."
---
# Logging in a Distributed System

A distributed system is composed of several applications calling each other to complete one operation. Each of these applications emits its own logs and often stores them in different locations. With distributed systems, logging is the easy part. Making sense of this large volume of logs from a logical point of view is much harder.

If you need to implement logging for a distributed system, the first step is to select a single location to store the logs. Dozens of solutions are available and PostSharp Logging is agnostic about this choice.

The next challenge is to correlate the log records coming from different applications to get a logical view of all records relevant to the processing of a specific request. Typically, this means adding HTTP headers on the client side and reading them on the server side. PostSharp Logging implements the [HTTP Correlation Protocol](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md) specification for HttpClient and ASP.NET. It also exposes APIs you can use to implement distributed logging with other technologies or implement different correlation protocols. 


## Preparing a set of applications for distributed logging


To prepare a set of applications for distributed logging:

1. If you have any application that emits outgoing HTTP requests or process incoming HTTP requests using ASP.NET or ASP.NET Core, instrument them as described in <xref:logging-instrumentation> and make sure to pass an implementation of the <xref:PostSharp.Patterns.Diagnostics.Correlation.ICorrelationProtocol> interface. 

    This will result in creating a property named `RequestId` on the client and the server part of the request. Additionally, the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> property on client and server side will be synchronized. 


2. If an application emits or accepts requests using a technology other than HttpClient or ASP.NET, for instance WCF or MSMQ, you have to implement correlation manually:

    a. On the client side, add a header to your outgoing request with the value of the current <xref:PostSharp.Patterns.Diagnostics.Contexts.LoggingContext.SyntheticId>, which you can get from `LoggingServices.DefaultBackend.CurrentContext.SyntheticId`. 

    b. On the server side, you need to read the parent id from the incoming headers, open a new activity (see <xref:log-custom-activities>), and set the <xref:PostSharp.Patterns.Diagnostics.OpenActivityOptions.SyntheticParentId> property. You may take this opportunity to implement support for per-request logging. See <xref:custom-logging-transactions> for details. 
<br>When this property is set, the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> will start with the value of the <xref:PostSharp.Patterns.Diagnostics.OpenActivityOptions.SyntheticParentId> property, and the in-process parent activity, if any, will be ignored. 


3. Choose a logging backend that supports logging properties (Serilog is a good choice) and configure it to write to a central log server such as Elasticsearch. You can correlate items using the `RequestId` property but it is more convenient to use the `EventId` property, generated from `SyntheticId`. With Serilog, include this property by adding <xref:PostSharp.Patterns.Diagnostics.Backends.Serilog.SerilogSpecialProperties.EventId> to the <xref:PostSharp.Patterns.Diagnostics.Backends.Serilog.SerilogLoggingBackendOptions.IncludedSpecialProperties> property. 


4. Choose a generation strategy for `SyntheticId` as described below. 


> [!WARNING]
> Enabling request correlation on a publicly available service is a potential security risk. Logging is often performed before the request is fully authenticated, so the content of the additional correlation HTTP headers cannot be trusted. However, they can be used to inject values into your logging system. These values may then be processed by your application, and will be stored in your log server.
>
> If you need to enable correlation of distributed logging on a public service, consider wrapping a default implementation of <xref:PostSharp.Patterns.Diagnostics.Correlation.ICorrelationProtocol> with your own logic that verifies the request security. 


## Using hierarchical context IDs

To correlate traces across processes, it is necessary to assign a proper identifier to each context and record. PostSharp Logging offers a suitable context identifier in the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> property. You can read this identifier from `logSource.CurrentContext.SyntheticId` or `activity.Context.SyntheticId`. 

The default implementation of the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> property respects the [Hierarchical Request-Id](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HierarchicalRequestId.md) specification. 

The most important benefit of this specification is that it is possible to select all children records of a logical distributed operation only by filtering by the beginning of the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId>, e.g. in pseudo-SQL: 

```sql
SELECT * FROM Records WHERE SyntheticId LIKE '|45ed51e.1.da4e9679%';
```

Additionally, our implementation generates sortable identifiers, so that ordering records by <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> makes sense: 

```sql
SELECT * FROM Records WHERE SyntheticId LIKE '|45ed51e.1.da4e9679%' ORDER BY SyntheticId ASC;
```

The filtering and ordering characteristics of the identifiers depend on the generation strategy, as explained below.


### Comparing generation strategies

The generation strategy of the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> property depends on several configuration properties: <xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions.ContextIdGenerationStrategy>, <xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions.RootSyntheticId>, and <xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions.SyntheticIdFormatter>. 

<xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions.ContextIdGenerationStrategy> is the most important configuration option. It determines how the <xref:PostSharp.Patterns.Diagnostics.Contexts.LoggingContext.Id> property (a 64-bit integer) is generated. There are two strategies: global and hierarchical. The default strategy, `LazyGlobal`, is a variant of the global strategy that generates the identifier the first time it is read. The two strategies are compared in the following table. 

|  | Global Strategy | Hierarchical Strategy |
|--|-----------------|-----------------------|
| Description | This strategy uses a single AppDomain-wide static 64-bit counter to generate the identifier. The <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> is therefore composed of the <xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions.RootSyntheticId> (by default, a random 64-bit integer) followed by the 64-bit <xref:PostSharp.Patterns.Diagnostics.Contexts.LoggingContext.Id> itself.  | The <xref:PostSharp.Patterns.Diagnostics.Contexts.LoggingContext.Id> property is generated using a counter in the scope of the parent context. If there is no parent context, the global strategy is used. Therefore, the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> property is composed of the <xref:PostSharp.Patterns.Diagnostics.Contexts.ILoggingContext.SyntheticId> of the parent context, plus the <xref:PostSharp.Patterns.Diagnostics.Contexts.LoggingContext.Id> of the current context.  |
| Length of generated identifiers | Short. | Potentially very long. |
| Performance | May cause thread contention on highly loaded systems because several threads may be trying to get exclusive access to the global counter at the same time. | No thread contention issue, but more CPU time required to render the id. |
| Ordering | Preserving time, but not causality (parallel async calls are mixed). | Preserving causality, but not time (parallel async calls are separated from each other). |
| Filtering | Only based on cross-process boundaries (see the <xref:PostSharp.Patterns.Diagnostics.OpenActivityOptions.SyntheticParentId> property in the procedure above).  | Based on any context (external activity, method, custom in-process activity). |


