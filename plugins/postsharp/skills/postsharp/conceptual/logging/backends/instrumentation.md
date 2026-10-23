---
uid: logging-instrumentation
redirect_from:
  - logging/logging-instrumentation
title: "Instrumentation of ASP.NET and HttpClient"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp Logging offers packages that support specific software stacks like ASP.NET Core, legacy ASP.NET, and HttpClient. It allows dynamic logging verbosity changes and distributed transaction tracking."
---
# Instrumentation of ASP.NET and HttpClient

PostSharp Logging is not only able to collect events from several logging frameworks, but it also comes with packages that directly support specific software stacks. Currently, three are supported:


## ASP.NET Core

If you are using ASP.NET Core, PostSharp Logging allows you to change the verbosity of logging dynamically, with or without source code, depending on the parameters of the incoming web requests. It also properly interprets the headers set by the `HttpClient` for distributed transaction tracking. 

To enable per-request logging with ASP.NET Core:

* Install the `PostSharp.Patterns.Diagnostics.AspNetCore` package into your top-level project. 

* In your `Program.Main` method, include a call to the <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetCore.AspNetCoreLogging.Initialize(PostSharp.Patterns.Diagnostics.Correlation.ICorrelationProtocol,System.Predicate{Microsoft.AspNetCore.Http.HttpRequest},PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata)> method. 
    If you want to enable support for distributed logging, you need to pass a value to the `correlationProtocol` parameter. The only available implementation is currently <xref:PostSharp.Patterns.Diagnostics.Correlation.LegacyHttpCorrelationProtocol>. 

[!code-csharp[](../../../code/samples/Logging/AspNetCore/LoggingInstrumentationAspNetCore.cs#example)]

This will log any incoming HTTP request and, if you have set up request correlation, it will interpret the HTTP headers related to distributed logging. To learn how to configure verbosity on a per-request basis, see <xref:log-enabling> for details. 


## Legacy ASP.NET

PostSharp Logging comes with an HTTP module that integrates with the legacy ASP.NET and IIS.

To enable per-request logging with ASP.NET:

* Install the `PostSharp.Patterns.Diagnostics.AspNetFramework` package into your top-level project. 

* Add <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetFramework.PostSharpLoggingHttpModule> as an HTTP module. 
    ```xml
    <configuration>
    	<system.webServer>
    		<modules>
    		  <add name="PostSharpLogging" type="PostSharp.Patterns.Diagnostics.Adapters.AspNetFramework.PostSharpLoggingHttpModule"/>
    		</modules>
    	</system.webServer>
    </configuration>
    ```


* If you want to enable support for distributed logging, you need to set the <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetFramework.PostSharpLoggingHttpModule.CorrelationProtocol> property during your application startup. The only available implementation is currently <xref:PostSharp.Patterns.Diagnostics.Correlation.LegacyHttpCorrelationProtocol>. 
    Therefore, add this to your `Global.asax`: 
    <!-- not-compiled: ASP.NET Framework only (Global.asax) -->
    ```csharp
    // Defines a filter that selects trusted requests.
    // Enabling HTTP Correlation Protocol for communication with untrusted devices is a security risk.
    Predicate<CorrelationRequest> trustedRequests = request => request.RemoteHost == "localhost" ||
                                                              request.RemoteHost == "127.0.0.1" ||
                                                              request.RemoteHost == "::1";

    // Assigns the HTTP Correlation Protocol to the HTTP module.
    PostSharpLoggingHttpModule.CorrelationProtocol = new LegacyHttpCorrelationProtocol( trustedRequests );
    ```


This will log any incoming HTTP request and, if you have set up request correlation, it will interpret the HTTP headers related to distributed logging. To learn how to configure verbosity on a per-request basis, see <xref:log-enabling> for details. 


## HttpClient

You can configure PostSharp Logging to instrument the .NET `HttpClient` class. This instrumentation will create an activity for any `HttpClient` operation and will properly set the headers so that distributed transactions can be tracked. It works together with the ASP.NET instrumentation. 

To add the `HttpClient` instrumentation to your code: 

* Install the `PostSharp.Patterns.Diagnostics.HttpClient` package into your top-level project. 

* In your `Program.Main` method, include a call to the <xref:PostSharp.Patterns.Diagnostics.Adapters.HttpClient.HttpClientLogging.Initialize(PostSharp.Patterns.Diagnostics.Correlation.ICorrelationProtocol,System.Predicate{System.Uri})> method. 
    If you want to enable support for distributed logging, you need to pass a value to the `correlationProtocol` parameter. The only available implementation is currently <xref:PostSharp.Patterns.Diagnostics.Correlation.LegacyHttpCorrelationProtocol>. It is important that you use the same protocol in the client and the server. 

[!code-csharp[](../../../code/samples/Logging/Integrations/LoggingInstrumentationHttpClient.cs#example)]

## See Also

**Other Resources**

<xref:distributed-logging>

