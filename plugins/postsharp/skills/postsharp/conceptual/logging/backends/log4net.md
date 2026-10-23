---
uid: log4net
redirect_from:
  - logging/log4net
title: "Logging with log4net"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging and log4net together, including steps to send PostSharp Logging events to log4net appenders and collect log4net events into PostSharp Logging."
---
# Logging with log4net

This article shows how to use PostSharp Logging and log4net together.


## Writing PostSharp Logging events to log4net


To send PostSharp Logging events to log4net appenders:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.Log4Net` package to your startup project. 


3. Create an XML file named `log4net.config`. In the file properties, set the **Copy to Output Directory** property to **Copy always**. 

    Add the following content to this file:

    [!code-xml[](../../../code/samples/Logging/Log4Net/LoggingLog4Net/log4net.config)]

    This example configuration file instructs log4net to write all log records to a file named `log4net.log` and to the console. 

    See the [log4net documentation](https://logging.apache.org/log4net/release/manual/configuration.html) for details about this configuration file. 


4. In the application startup file, include the following namespace imports:

    [!code-csharp[](../../../code/samples/Logging/Log4Net/LoggingLog4Net/LoggingLog4Net.cs#usings)]

    In the application startup method, include the following code:

    [!code-csharp[](../../../code/samples/Logging/Log4Net/LoggingLog4Net/LoggingLog4Net.cs#configure)]

    On .NET Core and .NET 5 or later, log4net does not support the default repository, so this code passes the repository explicitly.


If you run your application, you should now see a detailed log in a file named `log4net.log`. If your application is a console application, you should also see the log in the console, for instance:

[!code-text[](../../../code/samples/Logging/Log4Net/LoggingLog4Net/LoggingLog4Net.txt)]


## Collecting log4net events into PostSharp Logging

If your source code already emits log events using log4net, you can configure PostSharp Logging to collect log events directly emitted by the log4net API, so that they are all processed by the same PostSharp backend and log4net logger. One of the benefits is that indentation of manual logging and automatic logging will be synchronized. See <xref:log-collecting> for more information. 

Suppose that you have already configured log4net and PostSharp like this:

[!code-csharp[](../../../code/samples/Logging/Log4Net/Log4NetManualLogging/Log4NetManualLogging.cs#configure)]

[!code-csharp[](../../../code/samples/Logging/Log4Net/Log4NetManualLogging/Log4NetManualLogging.cs#manual)]

To collect log4net manual logging events into PostSharp Logging, replace that startup code with the following. You must do this before you create any log4net loggers: 

[!code-csharp[](../../../code/samples/Logging/Log4Net/Log4NetCollecting/Log4NetCollecting.cs#configure)]

You can now do standard log4net logging and the logging output will be collected and sent to PostSharp Logging:

[!code-csharp[](../../../code/samples/Logging/Log4Net/Log4NetCollecting/Log4NetCollecting.cs#manual)]

When this code runs in a logged method, the manual record is indented like the records of PostSharp Logging:

[!code-text[](../../../code/samples/Logging/Log4Net/Log4NetCollecting/Log4NetCollecting.txt)]

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.Log4Net.Log4NetLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.Log4Net.Log4NetCollectingRepositorySelector>

**Other Resources**

[Example project: PostSharp.Samples.Logging.Log4Net](https://samples.postsharp.net/f/PostSharp.Samples.Logging.Log4Net/)
<br><xref:log-collecting>

