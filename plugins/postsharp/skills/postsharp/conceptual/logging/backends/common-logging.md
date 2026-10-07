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

    ```csharp
    using Common.Logging;
    using Common.Logging.Simple;
    using PostSharp.Patterns.Diagnostics;
    using PostSharp.Patterns.Diagnostics.Backends.CommonLogging;
    ```

    In the application startup method, include the following code:

    ```csharp
    // Configure Common.Logging to direct outputs to the system console.
    LogManager.Adapter = new ConsoleOutLoggerFactoryAdapter();
    
    // Configure PostSharp Logging to direct outputs to Common.Logging.
    LoggingServices.DefaultBackend = new CommonLoggingLoggingBackend();
    ```

    This example code instructs Common.Logging to write all log records to the system console. See the [Common.Logging documentation](http://netcommon.sourceforge.net/docs/2.1.0/reference/html/ch01.html#logging-config) for details. 


If you run your application, you should also see the log in the console.

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.CommonLogging.CommonLoggingLoggingBackend>

**Other Resources**

[Example project: PostSharp.Samples.Logging.CommonLogging](https://samples.postsharp.net/f/PostSharp.Samples.Logging.CommonLogging/)

