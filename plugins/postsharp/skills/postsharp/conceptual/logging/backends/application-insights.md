---
uid: application-insights
redirect_from:
  - logging/application-insights
title: "Logging with Application Insights"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp Logging and Application Insights together, including setting up and configuring the services."
---
# Logging with Application Insights

This article shows how to use PostSharp Logging and Application Insights together.


To use PostSharp Logging with Application Insights:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. Add the `PostSharp.Patterns.Diagnostics.ApplicationInsights` package to your startup project. 


3. Set up your Application Insights resource [according to the official guide](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview#get-started). 


4. In the application startup method, include the following code before any logged code:

    [!code-csharp[](../../../code/samples/Logging/LoggingApplicationInsights.cs#configure)]

    > [!NOTE]
    > Instead of passing the instrumentation key to the constructor, you can use the parameterless constructor of <xref:PostSharp.Patterns.Diagnostics.Backends.ApplicationInsights.ApplicationInsightsLoggingBackend>. It reads the default Application Insights configuration, for example the environment variable `APPINSIGHTS_INSTRUMENTATIONKEY` or, on .NET Framework, an `ApplicationInsights.config` file.


## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.ApplicationInsights.ApplicationInsightsLoggingBackend>

