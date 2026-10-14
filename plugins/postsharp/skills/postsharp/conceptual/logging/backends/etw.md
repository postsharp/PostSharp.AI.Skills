---
uid: etw
redirect_from:
  - logging/etw
title: "Logging with ETW"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging and Event Tracing for Windows (ETW) together, including targeting ETW with PostSharp Logging and listening to ETW events using PerfView."
---
# Logging with ETW

This article shows how to use PostSharp Logging and Event Tracing for Windows (ETW) together.


## Targeting ETW


To target ETW with PostSharp Logging:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.Tracing` package to your startup project. 


3. In the application startup file, include the following namespace imports:

    [!code-csharp[](../../../code/samples/Logging/LoggingEtw.cs#usings)]

    In the application startup method, include the following code:

    [!code-csharp[](../../../code/samples/Logging/LoggingEtw.cs#configure)]


As a result of this procedure, PostSharp Logging will emit records to the ETW event source named `PostSharp-Patterns-Diagnostics`. You now need to attach a listener to this event source. 


## Listening to ETW events

ETW is a complex system, and explaining it is beyond the scope of this documentation. This section shows only how to collect and view ETW events on your development machine.

The procedure uses a tool named PerfView, developed by Microsoft.

To collect and view PostSharp Logging logs using PerfView:

1. Download PerfView from [the official Microsoft website](https://www.microsoft.com/en-us/download/details.aspx?id=28567). 


2. Before starting your application, execute the following command line:

    ```text
    perfview.exe collect -OnlyProviders *PostSharp-Patterns-Diagnostics
    ```


3. Execute your application.


4. In PerfView, click **Stop collecting**, then in the PerfView tree view click on **PerfViewData.etl.zip** and finally **Events**. 


5. To get a coherent view of all trace events, select all event types of the `PostSharp-Patterns-Diagnostics` category at the same time. 


## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.EventSource.EventSourceLoggingBackend>

**Other Resources**

[Example project: PostSharp.Samples.Logging.Etw](https://samples.postsharp.net/f/PostSharp.Samples.Logging.Etw/)

