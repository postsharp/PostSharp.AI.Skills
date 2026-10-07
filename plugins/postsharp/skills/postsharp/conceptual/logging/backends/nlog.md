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

    ```csharp
    using NLog;
    using NLog.Config;
    using NLog.Targets;
    using PostSharp.Patterns.Diagnostics;
    using PostSharp.Patterns.Diagnostics.Backends.NLog;
    ```

    In the application startup method, include the following code:

    ```csharp
    // Configure NLog.
    var nlogConfig = new LoggingConfiguration();
    
    var fileTarget = new FileTarget("file")
    {
        FileName = "nlog.log",
        KeepFileOpen = true,
        ConcurrentWrites = false,
    };
    
    nlogConfig.AddTarget(fileTarget);
    nlogConfig.LoggingRules.Add(new LoggingRule("*", NLog.LogLevel.Debug, fileTarget));
    
    var consoleTarget = new ConsoleTarget("console");
    nlogConfig.AddTarget(consoleTarget);
    nlogConfig.LoggingRules.Add(new LoggingRule("*", NLog.LogLevel.Debug, consoleTarget));
    
    LogManager.EnableLogging();
    
    
    // Configure PostSharp Logging to use NLog.
    LoggingServices.DefaultBackend = new NLogLoggingBackend(new LogFactory(nlogConfig));
    ```

    This example code instructs NLog to write all log records to a file named `nlog.log` and to the console. If you prefer, you can configure NLog with a configuration file. See the [NLog documentation](https://github.com/NLog/NLog/wiki/Tutorial#configuration) for details. 


If you run your application, you should now see a detailed log in a file named `nlog.log`. If your application is a console application, you should also see the log in the console. 


## Collecting NLog events into PostSharp Logging

If your source code already emits log events using NLog, you can configure PostSharp Logging to collect log events directly emitted by the NLog API, so that they are all processed by the same PostSharp backend and NLog logger. One of the benefits is that indentation of manual logging and automatic logging will be synchronized. See <xref:log-collecting> for more information. 

Suppose that you have already configured NLog and PostSharp like this:

```csharp
// Configure NLog
var configuration = new XmlLoggingConfiguration("nlog.config");

LogManager.Configuration = configuration; // Set it as the default configuration
LogManager.EnableLogging();

// Configure PostSharp Logging to use NLog
LoggingServices.DefaultBackend = new NLogLoggingBackend(new LogFactory(configuration));

// Emit manual log records. Note that this logger will skip PostSharp Logging, so indentation
// will not be respected.
NLog.Logger logger = LogManager.GetCurrentClassLogger();
logger.Info().Message( "Hello, {color} sky!", "blue" ).Write();
```

To collect NLog manual logging events into PostSharp, replace that startup code with the following:

```csharp
// Set up NLog to send all logging events to the PostSharp collector:
var target = new NLogCollectingTarget("toPostSharp"); // the name does not matter
var sourceConfiguration = new LoggingConfiguration();
sourceConfiguration.AddTarget( target ); 
sourceConfiguration.LoggingRules.Add( new LoggingRule("*", global::NLog.LogLevel.Trace, target) ); // Capture all events.
LogManager.Configuration = sourceConfiguration; // Set it as the default configuration
LogManager.EnableLogging();

// Create a separate NLog configuration that contains the targets for your final output. 
// If you have it in a configuration file, you can load it with:
var outputConfiguration = new XmlLoggingConfiguration("nlog.config");

// Use this final configuration to create a PostSharp backend and set it as the default backend:
LoggingServices.DefaultBackend = new NLogLoggingBackend(new LogFactory(outputConfiguration));
```

You can now do standard NLog logging and the logging output will be collected and sent to PostSharp Logging:

```csharp
NLog.Logger logger = LogManager.GetCurrentClassLogger();
logger.Info().Message( "Hello, {color} sky!", "blue" ).Write();
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.NLog.NLogLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.NLog.NLogCollectingTarget>

**Other Resources**

[Example project: PostSharp.Samples.Logging.NLog](https://samples.postsharp.net/f/PostSharp.Samples.Logging.NLog/)
<br><xref:log-collecting>

