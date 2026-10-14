---
uid: log-multiplexer
redirect_from:
  - logging/log-multiplexer
title: "Multiplexing Log Output to Several Frameworks"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp's MultiplexerBackend to send logging output to multiple logging frameworks simultaneously, including setting up, configuring backends, and handling logging errors."
---
# Multiplexing Log Output to Several Frameworks

This article shows how to send PostSharp Logging output to two or more logging frameworks using <xref:PostSharp.Patterns.Diagnostics.Backends.Multiplexer.MultiplexerBackend>. 


## Using the multiplexer

The <xref:PostSharp.Patterns.Diagnostics.Backends.Multiplexer.MultiplexerBackend> is a <xref:PostSharp.Patterns.Diagnostics.LoggingBackend> that sends logging events to two or more other logging backends ("child backends"). This way, you can log to multiple outputs or frameworks at the same time. For example, you could send output to console and Application Insights at the same time. 

To set it up, create two or more logging backends in your initialization code, pass them as arguments to the constructor of the multiplexer, and then use the multiplexer as your backend, for example:

[!code-csharp[](../../../code/samples/Logging/LogMultiplexer.cs#example)]

Each record is written twice, once by each backend:

[!code-text[](../../../code/samples/Logging/LogMultiplexer.txt)]


## Configuring the backends

You can still configure options, such as verbosity, logging of details and backend-specific options. You should do this configuration on the child backends, not the multiplexer backend itself.

If you attempt to set the minimum verbosity for a role, namespace or type on the multiplexer itself, a `NotSupportedException` is thrown. Any other options you set on the multiplexer (for example, <xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions.IncludeManualLoggingSourceLineInfo>) have no effect. Set them on the child backends instead. 

For example, you can configure one backend to receive all logging events and a second backend to receive only warnings like this:

[!code-csharp[](../../../code/samples/Logging/LogMultiplexerVerbosity.cs#example)]

The second backend only writes the warning:

[!code-text[](../../../code/samples/Logging/LogMultiplexerVerbosity.txt)]


## Handling logging errors

If an exception occurs in logging, only the <xref:PostSharp.Patterns.Diagnostics.LoggingBackend> responsible for the failure is disabled. The other backends in the multiplexer will continue to log events. 

If you need a different behavior (for example, you need to disable all logging backends in that case, or conversely, you do not want any backends disabled), you can override this default exception handling behavior. See <xref:logging-exception-handling> for details. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.Multiplexer.MultiplexerBackend>

**Other Resources**

<xref:logging-exception-handling>

