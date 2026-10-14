---
uid: logging-verbosity-api
redirect_from:
  - logging/logging-verbosity-api
title: "Adjusting Logging Verbosity Programmatically"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions on how to adjust logging verbosity programmatically using PostSharp Logging API, enabling/disabling with the backend API, and optimizing execution time when logging is disabled."
---
# Adjusting Logging Verbosity Programmatically

If configuring verbosity using an XML file does not fit your scenario, you can do it using normal API calls. This article describes how.


## Enabling/disabling logging with the PostSharp Logging API

To tune the logging verbosity for a specific type or namespace, first get the default <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration> by evaluating the <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.DefaultVerbosity> property, then use one of the <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration.SetMinimalLevel*>, <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration.SetMinimalLevelForType*> or <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration.SetMinimalLevelForNamespace*> method. 

The default logging level is `Debug`. 


### Example

The following code will cause PostSharp to log only exceptions for the `MyCompany` namespace and at the same time all messages for the `MyCompany.BusinessLayer` namespace. The order of the two lines of code is important. 

[!code-csharp[](../../code/samples/Logging/LoggingVerbosityApi.cs#namespaces)]

When the program calls a logged method of `MyCompany.Data` and then a logged method of `MyCompany.BusinessLayer`, only the second call is logged:

[!code-text[](../../code/samples/Logging/LoggingVerbosityApi.txt)]


## Configuring verbosity for the current execution context only

In the above section, you have configured verbosity through the <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.DefaultVerbosity> property. This property gives you access to the configuration of verbosity for the *default execution context* only. However, you can override verbosity for any execution context, for instance for a specific web request. 


To selectively enable logging for the current execution context:

1. Disable detailed logging at global level.

    [!code-csharp[](../../code/samples/Logging/LoggingVerbosityContext.cs#global)]


2. Create a <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration> by calling the <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.CreateVerbosityConfiguration> method. This <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration> will be used by all requests where you want to enable logging. Store it in a field. Note that the <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration> is a type that uses a large amount of memory, so you should avoid creating one instance for every request. 

    By default, the verbosity is set to `Debug`. You can configure the <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration> as needed, for instance using <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration.SetMinimalLevelForNamespace(PostSharp.Patterns.Diagnostics.LogLevel,System.String)>. 

    [!code-csharp[](../../code/samples/Logging/LoggingVerbosityContext.cs#create)]


3. Intercept the request before it is passed to your application code (typically using an ASP.NET Action Filter or a WCF Behavior), decide whether the request should be logged, and call the <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration.Use> method. Dispose the returned token once the request processing has completed. 

    [!code-csharp[](../../code/samples/Logging/LoggingVerbosityContext.cs#use)]



## Enabling/disabling with the backend API

Several logging frameworks offer a configuration mechanism that allows you to enable or disable logging. For a PostSharp log record to be emitted, two conditions need to be met: logging must be enabled by PostSharp (see above) and by the backend logging framework.

Logging frameworks generally have a concept of a *category* or of a *source* (the terminology can vary), which is typically determined by a string (usually a type name or a namespace). The corresponding concept in PostSharp Logging is the <xref:PostSharp.Patterns.Diagnostics.LoggingTypeSource> class, which is determined by two strings: a role (see <xref:PostSharp.Patterns.Diagnostics.LoggingRoles>) and a type name or namespace. 

By default, a <xref:PostSharp.Patterns.Diagnostics.LoggingTypeSource> is mapped to a category according to the full type name. If you want to control logging using the backend framework, you will need to customize this mapping. This can be done by setting a property of the <xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions> class. This property is different for each backend framework. 

Therefore, you enable or disable logging using the facilities implemented by the backend logging framework, using the full type name as the source or category name.


## Optimizing the execution time when logging is disabled

By default, PostSharp generates code that allows you to dynamically enable or disable logging for a specific type and severity. However, even if logging is disabled, your CPU still needs to execute the code that evaluates whether logging is enabled.

By setting the <xref:PostSharp.Patterns.Diagnostics.LoggingProfile.AllowDynamicEnabling> property of the logging profile to `false`, you can ask PostSharp to generate instructions that can be fully eliminated by the JIT compiler when logging is disabled. Therefore, the cost of inactive logging will be close to zero. Note that our latest tests show that the JIT compiler still emits instructions when logging is disabled, but it emits the equivalent of `if ( false ) { Log; }`, which has a very low performance overhead because of branch prediction. 

When you set the <xref:PostSharp.Patterns.Diagnostics.LoggingProfile.AllowDynamicEnabling> property to `false`, you need to configure the <xref:PostSharp.Patterns.Diagnostics.LoggingVerbosityConfiguration> object when the application initializes. Any change made after a logged type is JIT-compiled will be ignored for this specific type. 


### Example

The following `postsharp.config` enables the JIT-compiler optimizations for the default profile. 

[!code-xml[](../../code/samples/Logging/LoggingJitOptimization/postsharp.config)]


