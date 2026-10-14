---
uid: common-logging
redirect_from:
  - logging/common-logging
title: "Logging with Common.Logging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging with Common.Logging, including adding necessary packages and configuring output directions."
---
# Logging with Common.Logging

This article shows how to use PostSharp Logging and Common.Logging together.


To use PostSharp Logging with Common.Logging:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.CommonLogging` package to your startup project. 


3. In the application startup file, include the following namespace imports:

    [!code-csharp[](../../../code/samples/Logging/LoggingCommonLogging.cs#usings)]

    In the application startup method, include the following code:

    [!code-csharp[](../../../code/samples/Logging/LoggingCommonLogging.cs#configure)]

    This example code instructs Common.Logging to write all log records to the system console. See the [Common.Logging documentation](http://netcommon.sourceforge.net/docs/2.1.0/reference/html/ch01.html#logging-config) for details. 
    Common.Logging for .NET Standard does not include a console adapter. `ConsoleLoggerFactoryAdapter` is a small class of this example:


    [!code-csharp[](../../../code/samples/Logging/LoggingCommonLogging.cs#adapter)]


If you run your application, you should see the log in the console, for instance:

[!code-text[](../../../code/samples/Logging/LoggingCommonLogging.txt)]

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.CommonLogging.CommonLoggingLoggingBackend>

**Other Resources**

[Example project: PostSharp.Samples.Logging.CommonLogging](https://samples.postsharp.net/f/PostSharp.Samples.Logging.CommonLogging/)

