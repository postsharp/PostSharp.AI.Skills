---
uid: logging-aspnetcore
redirect_from:
  - logging/logging-aspnetcore
title: "Logging with ASP.NET Core"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging with ASP.NET Core. It details steps for writing PostSharp Logging events to ASP.NET Core and collecting ASP.NET Core logging events into PostSharp Logging."
---
# Logging with ASP.NET Core

This article shows how to use PostSharp Logging and ASP.NET Core logging together.


## Writing PostSharp Logging events to ASP.NET Core


To send PostSharp Logging events to the logging system of ASP.NET Core:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.Microsoft` package to your startup project. 


3. In *Program.cs*, set up ASP.NET Core logging like this: 

    [!code-csharp[](../../../code/samples/Logging/LoggingAspNetCore/LoggingAspNetCore.cs#logging)]


4. In *appsettings.json* and *appsettings.Development.json*, reduce the minimum logged `LogLevel` to `Debug`. PostSharp outputs most traces at the `Debug` level: 

    [!code-json[](../../../code/samples/Logging/LoggingAspNetCore/appsettings.json)]


5. After you build the application, and before you run it, set <xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft.MicrosoftLoggingBackend> as the default backend: 

    [!code-csharp[](../../../code/samples/Logging/LoggingAspNetCore/LoggingAspNetCore.cs#backend)]


Any methods annotated with <xref:PostSharp.Patterns.Diagnostics.LogAttribute> will now emit log messages into ASP.NET Core logging, as in this example:

```text
dbug: WebSite1.Pages.RazorPage1[2]
      RazorPage1.HelloWorld() | Starting.
dbug: WebSite1.Pages.RazorPage1[4]
      RazorPage1.HelloWorld() | Succeeded.
```


## Collecting ASP.NET Core logging events into PostSharp Logging

If your source code already emits log events using the <xref:Microsoft.Extensions.Logging.ILogger> interface of .NET Core, you can configure PostSharp Logging to collect log events directly emitted by <xref:Microsoft.Extensions.Logging.ILogger>, so that they are all processed by the same PostSharp backend and logger. One of the benefits is that indentation of manual logging and automatic logging will be synchronized. See <xref:log-collecting> for more information. 


To collect .NET Core manual logging events into PostSharp:

1. Add the `PostSharp.Patterns.Diagnostics.Microsoft` package to your startup project. 


2. In *Program.cs*, after you create the `WebApplicationBuilder`, add the following code: 

    [!code-csharp[](../../../code/samples/Logging/LoggingAspNetCoreCollecting.cs#collecting)]


Now, you can use <xref:PostSharp.Patterns.Diagnostics.LogAttribute> to add automatic logging and <xref:Microsoft.Extensions.Logging.ILogger> loggers to add manual log messages. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft.MicrosoftLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft.MicrosoftLoggingCollectingLoggerProvider>

**Other Resources**

<xref:log-collecting>

