---
uid: logging-trace
redirect_from:
  - logging/logging-trace
title: "Logging with System.Diagnostics.Trace"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging with System.Diagnostics.Trace, including sending PostSharp Logging events to Trace listeners and collecting Trace events into PostSharp Logging."
---
# Logging with System.Diagnostics.Trace

This article shows how to use PostSharp Logging and `System.Diagnostics.Trace` together. 


## Writing PostSharp Logging events to System.Diagnostics.Trace


To send PostSharp Logging events to Trace listeners:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.Tracing` package to your startup project. 


3. In the application startup method, include the following code:

    ```csharp
    LoggingServices.DefaultBackend = new TraceLoggingBackend();
    ```



## Collecting Trace events into PostSharp Logging

If your source code already emits log events using `System.Diagnostics.Trace`, you can configure PostSharp Logging to collect log events directly emitted by `System.Diagnostics.Trace`, so that they are all processed by the same PostSharp backend and `System.Diagnostics.Trace` logger. One of the benefits is that indentation of manual logging and automatic logging will be synchronized. See <xref:log-collecting> for more information. 

Suppose that you have already configured `System.Diagnostics.Trace` and PostSharp like this: 

```csharp
// Add additional listeners:
Trace.Listeners.Add( new ConsoleTraceListener() );

// Configure PostSharp Logging to use Trace:
LoggingServices.DefaultBackend = new TraceLoggingBackend();

// Emit manual log records. Note that this logger will skip PostSharp Logging, so indentation
// will not be respected.
Trace.TraceError( "The {0} sky is shining {1}!", "blue", "bright" );
```

To collect Trace manual logging events into PostSharp, replace that startup code with the following:

```csharp
// Add additional listeners:
Trace.Listeners.Add( new ConsoleTraceListener() );

// Then capture all trace listeners and remove them:
var listeners = TraceCollectingTraceListener.RedirectLoggingToPostSharp();

// Then create a PostSharp backend and pass the captured trace listeners to the backend:
LoggingServices.DefaultBackend = new TraceLoggingBackend( listeners );
```

You can now do standard Trace logging and the logging output will be collected and sent to PostSharp Logging:

```csharp
Trace.TraceError( "The {0} sky is shining {1}!", "blue", "bright" );
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.Trace.TraceLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.Trace.TraceCollectingTraceListener>

**Other Resources**

<xref:log-collecting>

