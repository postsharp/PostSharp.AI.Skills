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

    ```xml
    <log4net>
      <appender name="file" type="log4net.Appender.FileAppender">
    
        <file value="log4net.log" />
    
        <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="%date [%thread] %-5level %logger %ndc - %message%newline" />
        </layout>
      </appender>
    
      <appender name="console" type="log4net.Appender.ColoredConsoleAppender">
        <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="%date [%thread] %-5level %logger [%property{NDC}] - %message%newline" />
        </layout>
      </appender>
    
      <root>
        <level value="DEBUG" />
        <appender-ref ref="file" />
        <appender-ref ref="console" />
      </root>
    
    </log4net>
    ```

    This example configuration file instructs log4net to write all log records to a file named `log4net.log` and to the console. 

    See the [log4net documentation](https://logging.apache.org/log4net/release/manual/configuration.html) for details about this configuration file. 


4. In the application startup file, include the following namespace imports:

    ```csharp
    using log4net.Config;
    using PostSharp.Patterns.Diagnostics;
    using PostSharp.Patterns.Diagnostics.Backends.Log4Net;
    ```

    In the application startup method, include the following code:

    ```csharp
    // Configure log4net
    XmlConfigurator.Configure(new FileInfo("log4net.config"));
    
    // Configure PostSharp Logging to use log4net
    LoggingServices.DefaultBackend = new Log4NetLoggingBackend();
    ```


If you run your application, you should now see a detailed log in a file named `log4net.log`. If your application is a console application, you should also see the log in the console. 


## Collecting log4net events into PostSharp Logging

If your source code already emits log events using log4net, you can configure PostSharp Logging to collect log events directly emitted by the log4net API, so that they are all processed by the same PostSharp backend and log4net logger. One of the benefits is that indentation of manual logging and automatic logging will be synchronized. See <xref:log-collecting> for more information. 

Suppose that you have already configured log4net and PostSharp like this:

```csharp
// Configure log4net
XmlConfigurator.Configure(new FileInfo("log4net.config"));

// Configure PostSharp Logging to use Log4Net
LoggingServices.DefaultBackend = new Log4NetLoggingBackend();

// Emit manual log records. Note that this logger will skip PostSharp Logging, so indentation
// will not be respected.
var logger = LogManager.GetLogger( typeof(HelloWorldClass) );
logger.Warn( "Hello, world." );
```

To collect log4net manual logging events into PostSharp Logging, replace that startup code with the following. You must do this before you create any log4net loggers: 

```csharp
// This sets the PostSharp repository selector as the active log4net repository selector.
// All loggers created from log4net static methods, and all logger repositories created from
// now on will come from this repository selector. 
//
// This repository selector creates loggers that send all logging events to PostSharp Logging.
//
// The relay repository returned by the RedirectLoggingToPostSharp method creates loggers that 
// are *not* redirected to PostSharp Logging and it serves as the repository for your final output loggers.
ILoggerRepository relay = Log4NetCollectingRepositorySelector.RedirectLoggingToPostSharp();

// Configure the *relay* repository (instead of the default repository) with your final output appenders:
XmlConfigurator.Configure(relay, new FileInfo("log4net.config"));

// Use the relay repository to create a Log4NetLoggingBackend and set it as the default backend:
LoggingServices.DefaultBackend = new Log4NetLoggingBackend(relay);
```

You can now do standard log4net logging and the logging output will be collected and sent to PostSharp Logging:

```csharp
var logger = LogManager.GetLogger( typeof(HelloWorldClass) );
logger.Warn( "Hello, world." );
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.Log4Net.Log4NetLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.Log4Net.Log4NetCollectingRepositorySelector>

**Other Resources**

[Example project: PostSharp.Samples.Logging.Log4Net](https://samples.postsharp.net/f/PostSharp.Samples.Logging.Log4Net/)
<br><xref:log-collecting>

