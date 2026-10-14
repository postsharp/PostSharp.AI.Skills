---
uid: nlog
redirect_from:
  - logging/nlog
title: "Logging with NLog"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging and NLog together, including how to write PostSharp Logging events to NLog and collect NLog events into PostSharp Logging."
---
# Logging with NLog

This article shows how to use PostSharp Logging and NLog together.


## Writing PostSharp Logging events to NLog


To send PostSharp Logging events to NLog targets:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.NLog` package to your startup project. 


3. In the application startup file, include the following namespace imports:

    [!code-csharp[](../../../code/samples/Logging/LoggingNLog.cs#usings)]

    In the application startup method, include the following code:

    [!code-csharp[](../../../code/samples/Logging/LoggingNLog.cs#configure)]

    This example code instructs NLog to write all log records to a file named `nlog.log` and to the console. If you prefer, you can configure NLog with a configuration file. See the [NLog documentation](https://github.com/NLog/NLog/wiki/Tutorial#configuration) for details. 


If you run your application, you should now see a detailed log in a file named `nlog.log`. If your application is a console application, you should also see the log in the console, for instance:

[!code-text[](../../../code/samples/Logging/LoggingNLog.txt)]


## Collecting NLog events into PostSharp Logging

If your source code already emits log events using NLog, you can configure PostSharp Logging to collect log events directly emitted by the NLog API, so that they are all processed by the same PostSharp backend and NLog logger. One of the benefits is that indentation of manual logging and automatic logging will be synchronized. See <xref:log-collecting> for more information. 

Suppose that you have already configured NLog and PostSharp like this:

[!code-csharp[](../../../code/samples/Logging/NLogManualLogging/NLogManualLogging.cs#configure)]

[!code-csharp[](../../../code/samples/Logging/NLogManualLogging/NLogManualLogging.cs#manual)]

To collect NLog manual logging events into PostSharp, replace that startup code with the following:

[!code-csharp[](../../../code/samples/Logging/NLogCollecting/NLogCollecting.cs#configure)]

You can now do standard NLog logging and the logging output will be collected and sent to PostSharp Logging:

[!code-csharp[](../../../code/samples/Logging/NLogCollecting/NLogCollecting.cs#manual)]

When this code runs in a logged method, the manual record is indented like the records of PostSharp Logging:

[!code-text[](../../../code/samples/Logging/NLogCollecting/NLogCollecting.txt)]

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.NLog.NLogLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.NLog.NLogCollectingTarget>

**Other Resources**

[Example project: PostSharp.Samples.Logging.NLog](https://samples.postsharp.net/f/PostSharp.Samples.Logging.NLog/)
<br><xref:log-collecting>

