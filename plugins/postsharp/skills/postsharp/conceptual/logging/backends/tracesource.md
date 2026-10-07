---
uid: logging-tracesource
redirect_from:
  - logging/logging-tracesource
title: "Logging with System.Diagnostics.TraceSource"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging with System.Diagnostics.TraceSource, including sending PostSharp Logging events to TraceSource listeners and collecting TraceSource events into PostSharp Logging."
---
# Logging with System.Diagnostics.TraceSource

This article shows how to use PostSharp Logging and `System.Diagnostics.TraceSource` together. 


## Writing PostSharp Logging events to System.Diagnostics.TraceSource


To send PostSharp Logging events to TraceSource listeners:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.Tracing` package to your startup project. 


3. In the application startup method, include the following code:

    ```csharp
    LoggingServices.DefaultBackend = new TraceSourceLoggingBackend();
    ```


Note that by default, PostSharp Logging will use a different <xref:System.Diagnostics.TraceSource> for each logged type in source code, and the source will be named after the namespace-qualified type name. You can override this behavior using the following code: 

```csharp
LoggingServices.DefaultBackend = new TraceSourceLoggingBackend()
{
    Options =
    {
        GetTraceSourceName = ( source ) => "PostSharpLogging" // Change with your source name.
    }
};
```


## Collecting TraceSource events into PostSharp Logging

You can also configure `System.Diagnostics.TraceSource` to send manual logging events to targets via a PostSharp collector. This way, all logging events, including those created directly with TraceSource methods, are processed by the same PostSharp backend. See <xref:log-collecting> for more information. 




To collect TraceSource manual logging events into PostSharp:

1. Add the `PostSharp.Patterns.Diagnostics.Tracing` package to your startup project. 


2. Configure two trace sources in your `app.config` file, one that you will use in code, and one that controls the final output: 

    ```xml
    <configuration>  
      <system.diagnostics>  
        <sources>  
          <source name="ManualLogging"
            switchName="sourceSwitch"
            switchType="System.Diagnostics.SourceSwitch">  
            <listeners>  
              <add name="toPostSharp" type="PostSharp.Patterns.Diagnostics.Backends.TraceSource.TraceSourceCollectingTraceListener, PostSharp.Patterns.Diagnostics.Backends.Tracing" />
              <remove name="Default"/>  
            </listeners>  
          </source>  
          <source name="PostSharpLogging"
            switchName="sourceSwitch"
            switchType="System.Diagnostics.SourceSwitch">  
            <listeners>  
              <add name="console" type="System.Diagnostics.ConsoleTraceListener" />
              <remove name="Default"/>  
            </listeners>  
          </source>  
        </sources>  
        <switches>  
          <add name="sourceSwitch" value="Verbose"/>  
        </switches>  
      </system.diagnostics>  
    </configuration>
    ```


3. In the application startup method, include the following code:

    ```csharp
    LoggingServices.DefaultBackend = new TraceSourceLoggingBackend()
    {
        Options =
        {
            GetTraceSourceName = ( source ) => "PostSharpLogging"
        }
    };
    ```


4. You can now do standard `TraceSource` logging and the logging output will be collected and sent to PostSharp Logging. Use the `TraceSource` configured with <xref:PostSharp.Patterns.Diagnostics.Backends.TraceSource.TraceSourceCollectingTraceListener> to emit logging events: 

    ```csharp
    var ts = new TraceSource("ManualLogging");
    ts.TraceInformation( "Hello, world." );
    ```


Now, you can use <xref:PostSharp.Patterns.Diagnostics.LogAttribute> to add automatic logging and `TraceSource` methods to add manual log messages. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.TraceSource.TraceSourceLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.TraceSource.TraceSourceCollectingTraceListener>

**Other Resources**

<xref:log-collecting>

