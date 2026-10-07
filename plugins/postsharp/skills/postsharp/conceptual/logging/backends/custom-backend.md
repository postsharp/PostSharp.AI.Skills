---
uid: custom-logging-backend
redirect_from:
  - logging/custom-logging-backend
title: "Building Your Own Logging Backend (Adapter)"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to build a custom logging backend (adapter) for PostSharp Logging by overriding four classes: TextLoggingBackendOptions, LoggingTypeSource, TextLogRecordBuilder, and TextLoggingBackend."
---
# Building Your Own Logging Backend (Adapter)

The adapter between PostSharp Logging and the target logging framework is called a *backend* in PostSharp terminology. PostSharp comes with ready-made support for the most popular logging frameworks in .NET. If you need to integrate with a different logging solution, you can implement a custom logging backend. 

You must implement at least four classes to build a custom logging backend.


## Override the TextLoggingBackendOptions class

By design, every logging backend must implement a class exposing all available run-time options. This is a design decision to expose options on a different class than the backend class itself. Even if your custom backend does not expose new options, we suggest that you follow this convention and create a new empty class.


### Example

```csharp
using PostSharp.Patterns.Diagnostics.Backends;

namespace PostSharp.Samples.Logging.CustomBackend.ServiceStack
{
  public class ServiceStackLoggingBackendOptions : TextLoggingBackendOptions
  {
  }
}
```


## Override the LoggingTypeSource class

The <xref:PostSharp.Patterns.Diagnostics.LoggingTypeSource> class corresponds to the `ILog` or `ILogger` concept of several logging frameworks. Your <xref:PostSharp.Patterns.Diagnostics.LoggingTypeSource> will typically contain one field of this type, which will be initialized in the constructor. 

In addition to exposing the backend logger, the <xref:PostSharp.Patterns.Diagnostics.LoggingTypeSource> class exposes the <xref:PostSharp.Patterns.Diagnostics.LoggingTypeSource.IsBackendEnabled(PostSharp.Patterns.Diagnostics.LogLevel)> method, which determines whether logging is enabled for the specified level and the current type and role. 

Your implementation of <xref:PostSharp.Patterns.Diagnostics.LoggingTypeSource> must be immutable or thread-safe. 


### Example

```csharp
using PostSharp.Patterns.Diagnostics;
using ServiceStack.Logging;
using System;

namespace PostSharp.Samples.Logging.CustomBackend.ServiceStack
{
  public class ServiceStackLoggingTypeSource : LoggingTypeSource
  {
    public ServiceStackLoggingTypeSource(LoggingNamespaceSource parent, string name, Type sourceType)
      : base(parent, name, sourceType )
    {
      Log = LogManager.GetLogger(sourceType);
    }

    public ILog Log { get; }

    protected override bool IsBackendEnabled(LogLevel level)
    {
      switch (level)
      {
        case LogLevel.Trace:
        case LogLevel.Debug:
          return Log.IsDebugEnabled;

        default:
          return true;
      }
    }
  }
}
```


## Override the TextLogRecordBuilder class

The role of the <xref:PostSharp.Patterns.Diagnostics.RecordBuilders.TextLogRecordBuilder> is to create a string representing the current log record and finally to emit this string to the target logging framework. 

The <xref:PostSharp.Patterns.Diagnostics.RecordBuilders.TextLogRecordBuilder> class already contains the logic that creates the string. If you do not need to alter the string formatting logic, all you have to do is to implement the <xref:PostSharp.Patterns.Diagnostics.RecordBuilders.TextLogRecordBuilder.Write(PostSharp.Patterns.Formatters.UnsafeString)> method. 

If you need to customize the formatting of the string, or if you need to implement semantic logging, you will need to override other virtual methods of the <xref:PostSharp.Patterns.Diagnostics.RecordBuilders.TextLogRecordBuilder> class. See the API reference for details. 

Your implementation of the <xref:PostSharp.Patterns.Diagnostics.RecordBuilders.TextLogRecordBuilder> class does not need to be thread-safe. All threads involved in logging have their own instance of the <xref:PostSharp.Patterns.Diagnostics.RecordBuilders.TextLogRecordBuilder> class. 


### Example

```csharp
using PostSharp.Patterns.Diagnostics;
using PostSharp.Patterns.Diagnostics.Backends;
using PostSharp.Patterns.Diagnostics.RecordBuilders;
using PostSharp.Patterns.Formatters;

namespace PostSharp.Samples.Logging.CustomBackend.ServiceStack
{
  public class ServiceStackLogRecordBuilder : TextLogRecordBuilder
  {
    public ServiceStackLogRecordBuilder(TextLoggingBackend backend) : base(backend)
    {
    }

    protected override void Write(UnsafeString message)
    {
      var log = ((ServiceStackLoggingTypeSource) TypeSource).Log;
      var messageString = message.ToString();

      switch (Level)
      {
        case LogLevel.None:
          break;

        case LogLevel.Trace:
        case LogLevel.Debug:
          if (Exception == null)
          {
            log.Debug(messageString);
          }
          else
          {
            log.Debug(messageString, Exception);
          }

          break;

        case LogLevel.Info:
          if (Exception == null)
          {
            log.Info(messageString);
          }
          else
          {
            log.Info(messageString, Exception);
          }

          break;

        case LogLevel.Warning:
          if (Exception == null)
          {
            log.Warn(messageString);
          }
          else
          {
            log.Warn(messageString, Exception);
          }

          break;

        case LogLevel.Error:
          if (Exception == null)
          {
            log.Error(messageString);
          }
          else
          {
            log.Error(messageString, Exception);
          }

          break;

        case LogLevel.Critical:
          if (Exception == null)
          {
            log.Fatal(messageString);
          }
          else
          {
            log.Fatal(messageString, Exception);
          }

          break;
      }
    }
  }
}
```


## Override the TextLoggingBackend class

The root of a logging backend is the <xref:PostSharp.Patterns.Diagnostics.Backends.TextLoggingBackend> class. You can consider this class as a factory type that instantiates other facilities needed to log with your custom logging backend. 

Since the <xref:PostSharp.Patterns.Diagnostics.Backends.TextLoggingBackend> is an abstract class, you will have to implement several factory methods.

To implement the backend class:

1. Create a new class and derive it from the <xref:PostSharp.Patterns.Diagnostics.Backends.TextLoggingBackend> class. 


2. Add a get-only `Options` property and initialize it to a new instance of your options class. 


3. Implement the <xref:PostSharp.Patterns.Diagnostics.Backends.TextLoggingBackend.GetTextBackendOptions> abstract method so that it returns the value of your `Options` property. 


4. Implement the <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateTypeSource(PostSharp.Patterns.Diagnostics.LoggingNamespaceSource,System.Type)> abstract method so that it returns a new instance of your `LoggingTypeSource` class. 


5. Implement the <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateRecordBuilder> abstract method so that it returns a new instance of your `LogRecordBuilder` class. 



### Example

```csharp
using PostSharp.Patterns.Diagnostics;
using PostSharp.Patterns.Diagnostics.Backends;
using PostSharp.Patterns.Diagnostics.RecordBuilders;
using System;

namespace PostSharp.Samples.Logging.CustomBackend.ServiceStack
{
  public class ServiceStackLoggingBackend : TextLoggingBackend
  {
    public new ServiceStackLoggingBackendOptions Options { get; } = new ServiceStackLoggingBackendOptions();

    protected override LoggingTypeSource CreateTypeSourceBySourceName(LoggingNamespaceSource parent, string sourceName)
    {
      return new ServiceStackLoggingTypeSource(parent, sourceName, null);
    }

    protected override LoggingTypeSource CreateTypeSource(LoggingNamespaceSource parent, Type type)
    {
      return new ServiceStackLoggingTypeSource(parent, null, type);
    }

    public override LogRecordBuilder CreateRecordBuilder()
    {
      return new ServiceStackLogRecordBuilder(this);
    }

    protected override TextLoggingBackendOptions GetTextBackendOptions()
    {
      return Options;
    }
  }
}
```


## Overriding context classes

Contexts are typically used to represent the execution of a logged method and a custom activity. There are typically two log records per context: the entry record and the exit record. The role of context classes is to expose or store pieces of information that are shared by several records of the same context. Unless you need to store more pieces of information in the context, you do not need to extend the context classes. Most backend implementations do not extend context classes.

Context classes all derive from the <xref:PostSharp.Patterns.Diagnostics.Contexts.LoggingContext> class. The following table lists all context classes. If you choose to derive a context class, you will also need to override the corresponding factory method in your <xref:PostSharp.Patterns.Diagnostics.LoggingBackend> class. 

| Context class | Factory method | Description |
|---------------|----------------|-------------|
| <xref:PostSharp.Patterns.Diagnostics.Contexts.SyncMethodLoggingContext> | <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateSyncMethodContext(PostSharp.Patterns.Diagnostics.Contexts.ThreadLoggingContext)> | Normal method (neither `async` nor iterator). |
| <xref:PostSharp.Patterns.Diagnostics.Contexts.AsyncMethodLoggingContext> | <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateAsyncMethodContext> | `async` method. |
| <xref:PostSharp.Patterns.Diagnostics.Contexts.IteratorLoggingContext> | <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateIteratorContext> | Iterator method. |
| <xref:PostSharp.Patterns.Diagnostics.Contexts.SyncCustomActivityLoggingContext> | <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateSyncCustomActivityContext(PostSharp.Patterns.Diagnostics.Contexts.ThreadLoggingContext)> | Synchronous custom activity. |
| <xref:PostSharp.Patterns.Diagnostics.Contexts.AsyncCustomActivityLoggingContext> | <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateAsyncCustomActivityContext> | Asynchronous custom activity. |
| <xref:PostSharp.Patterns.Diagnostics.Contexts.ThreadLoggingContext> | <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateThreadContext> | A special kind of context that represents the roots of the context tree and contains all thread-static fields. |
| <xref:PostSharp.Patterns.Diagnostics.Contexts.EphemeralLoggingContext> | <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateEphemeralContext(PostSharp.Patterns.Diagnostics.Contexts.ThreadLoggingContext)> | A degenerate kind of context used when a log record is emitted out of a context, for instance when an exception record is logged without the corresponding entry record. |

## See Also

**Other Resources**

[Example project: PostSharp.Samples.Logging.CustomBackend.ServiceStack](https://samples.postsharp.net/f/PostSharp.Samples.Logging.CustomBackend.ServiceStack/)
<br>[Example project: PostSharp.Samples.Logging.Customization](https://samples.postsharp.net/f/PostSharp.Samples.Logging.Customization/)

