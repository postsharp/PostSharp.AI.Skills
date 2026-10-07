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


3. Set up ASP.NET Core logging in your `CreateHostBuilder` like this: 

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
      Host.CreateDefaultBuilder(args)
        .ConfigureLogging(loggingBuilder =>
        {
          loggingBuilder.ClearProviders();
          loggingBuilder.AddConsole();
        })
        .ConfigureWebHostDefaults(webBuilder => { webBuilder.UseStartup<Startup>(); });
    ```


4. In *appsettings.json* and *appsettings.Development.json*, reduce the minimum logged `LogLevel` to `Debug`. PostSharp outputs most traces at the `Debug` level: 

    ```json
    {
      "Logging": {
        "LogLevel": {
          "Default": "Debug",
          "Microsoft": "Warning",
          "Microsoft.Hosting.Lifetime": "Information"
        }
      }
    }
    ```


5. Set <xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft.MicrosoftLoggingBackend> as the default backend in your `Main` method: 

    ```csharp
    IHost host = CreateHostBuilder(args).Build();
    var loggerFactory = (ILoggerFactory) host.Services.GetService(typeof(ILoggerFactory));
    LoggingServices.DefaultBackend = new MicrosoftLoggingBackend(loggerFactory);
    host.Run();
    ```


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


2. Find your `CreateHostBuilder` method and add `ConfigureLogging`. All initialization code will go in there: 

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
      Host.CreateDefaultBuilder(args)
      .ConfigureLogging(loggingBuilder =>
        {
          // Logging initialization code goes here.
        })
      .ConfigureWebHostDefaults(webBuilder => { webBuilder.UseStartup<Startup>(); });
    ```


3. Replace that code with the following:

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
      Host.CreateDefaultBuilder(args)
      .ConfigureLogging(loggingBuilder =>
        {
          // First, set up your final logging providers. These are the providers, 
          // such as console or a log file, where you want the final output of logging to go:
          loggingBuilder.ClearProviders();
          loggingBuilder.AddConsole();
    	  
          // Next, capture those logging providers in a list and remove them:
          List<ILoggerProvider> loggerProviders =  loggingBuilder.Services.BuildServiceProvider().GetServices<ILoggerProvider>().ToList();
          loggingBuilder.ClearProviders();
    	  
          // Next, add the PostSharp logging provider which sends all events to PostSharp:
          loggingBuilder.AddPostSharp();
    	  
          // Finally, use the captured providers to create a PostSharp backend and set it as the default backend:
          LoggingServices.DefaultBackend = new MicrosoftLoggingBackend(new LoggerFactory(loggerProviders));
        })
      .ConfigureWebHostDefaults(webBuilder => { webBuilder.UseStartup<Startup>(); });
    ```


Now, you can use <xref:PostSharp.Patterns.Diagnostics.LogAttribute> to add automatic logging and <xref:Microsoft.Extensions.Logging.ILogger> loggers to add manual log messages. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft.MicrosoftLoggingBackend>
<br><xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft.MicrosoftLoggingCollectingLoggerProvider>

**Other Resources**

<xref:log-collecting>

