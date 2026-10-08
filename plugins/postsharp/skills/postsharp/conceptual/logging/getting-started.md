---
uid: add-logging
redirect_from:
  - logging/add-logging
title: "Getting Started with PostSharp Logging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on how to implement and configure PostSharp Logging in your codebase. It covers steps from adding logging to projects, choosing and configuring a logging framework, to setting logging verbosity."
---
# Getting Started with PostSharp Logging

When you work with a codebase, you often need to add logging, either as a non-functional requirement or to assist during the development process. In either situation, you will want to include information about the parameters passed to the method when it was called as well as the parameter values once the method call has completed. This can be a tedious and brittle process. As you work and refactor methods, the order and types of parameters may change, parameters may be added and some may be removed. Along with performing these refactorings, you have to remember to update the logging messages to keep them in sync. This is something that is easy to forget, and once forgotten, the output of the logging is much less useful.

Logging is an example of boilerplate code. Performing logging imperatively not only leads to the problems with refactoring mentioned above. It also makes your code harder to understand.

PostSharp offers a solution to all of these problems. PostSharp Logging allows you to configure where logging should be performed and takes over the task of keeping your log entries in sync as you add, remove and refactor your codebase. Using PostSharp Logging does not require changing your codebase, so you can keep the production code clearly understandable with no boilerplate code.

The following steps add trace logging for the start and completion of method calls.


## Step 1. Adding logging to your projects


### Adding logging to a specific project

1. Add a reference to the `PostSharp.Patterns.Diagnostics` package to your project. 


2. Create a source code file where you will add all project-wide aspects. We suggest naming this file `GlobalAspects.cs`. 

    Add the following content to this file:

    ```csharp
    using PostSharp.Patterns.Diagnostics;
    using PostSharp.Extensibility;
    
    [assembly: Log(AttributePriority = 1, AttributeTargetMemberAttributes = MulticastAttributes.Protected | MulticastAttributes.Internal | MulticastAttributes.Public)]
    [assembly: Log(AttributePriority = 2, AttributeExclude = true, AttributeTargetMembers = "get_*" )]
    ```

    This code adds logging to all methods except private methods and except property getters. You can edit this code to target methods relevant to your scenarios. See <xref:applying-aspects> for details. 


3. If there are several, but not many, projects in your solution, repeat this procedure for each project. Note that you can share the `GlobalAspects.cs` file among several projects. 


PostSharp will now add logging before and after the execution of all methods targeted by the logging aspect.


### Adding logging to all projects or a large number of projects

1. Add a reference to the `PostSharp.Patterns.Diagnostics` package to all projects where you want to use logging. 


2. Create a file named `postsharp.config` in a directory that contains all the projects where you want to use logging. These projects may be in subdirectories. See <xref:configuration-system>. 


3. Add the following content to the `postsharp.config` file: 

    ```xml
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
      <Multicast>
        <When Condition="{has-plugin('PostSharp.Patterns.Diagnostics')}">
          <LogAttribute xmlns="clr-namespace:PostSharp.Patterns.Diagnostics;assembly:PostSharp.Patterns.Diagnostics" />
        </When>
      </Multicast>
    </Project>
    ```

    That XML configuration file adds logging to all methods in all projects that have the `PostSharp.Patterns.Diagnostics` package. You can see more options at <xref:xml-multicasting>. 


PostSharp will now add logging before and after the execution of all methods targeted by the logging aspect.

The next step is to configure logging at run time. You should at least choose the logging framework that writes the records. We call this concept the *logging backend*. 


## Step 2. Choose your logging framework

The role of PostSharp Logging is to generate logging records, but PostSharp itself does not intend to write these logs to files, databases, or network services. Several excellent open-source projects and commercial services already fulfill this role. In PostSharp terminology, the target logging framework is called the logging *backend*. To see the logged records, you first need to choose and then configure a logging backend. 

PostSharp integrates with several logging frameworks directly. You can choose from the following implementations:

| Name | Class | Package |
|------|-------|---------------------------------------------|
| Microsoft Application Insights | <xref:PostSharp.Patterns.Diagnostics.Backends.ApplicationInsights.ApplicationInsightsLoggingBackend> | [PostSharp.Patterns.Diagnostics.ApplicationInsights](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.ApplicationInsights/) |
| Common.Logging | <xref:PostSharp.Patterns.Diagnostics.Backends.CommonLogging.CommonLoggingLoggingBackend> | [PostSharp.Patterns.Diagnostics.CommonLogging](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.CommonLogging/) |
| System.Console.WriteLine | <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleLoggingBackend> | [PostSharp.Patterns.Diagnostics](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics/) |
| ETW (System.Diagnostics.EventSource) | <xref:PostSharp.Patterns.Diagnostics.Backends.EventSource.EventSourceLoggingBackend> | [PostSharp.Patterns.Diagnostics.Tracing](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.Tracing/) |
| Log4Net | <xref:PostSharp.Patterns.Diagnostics.Backends.Log4Net.Log4NetLoggingBackend> | [PostSharp.Patterns.Diagnostics.Log4Net](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.Log4Net/) |
| Microsoft.Extensions.Logging | <xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft.MicrosoftLoggingBackend> | [PostSharp.Patterns.Diagnostics.Microsoft](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.Microsoft/) |
| NLog | <xref:PostSharp.Patterns.Diagnostics.Backends.NLog.NLogLoggingBackend> | [PostSharp.Patterns.Diagnostics.NLog](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.NLog/) |
| Serilog | <xref:PostSharp.Patterns.Diagnostics.Backends.Serilog.SerilogLoggingBackend> | [PostSharp.Patterns.Diagnostics.Serilog](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.Serilog/) |
| System.Diagnostics.Trace | <xref:PostSharp.Patterns.Diagnostics.Backends.Trace.TraceLoggingBackend> | [PostSharp.Patterns.Diagnostics.Tracing](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.Tracing/) |
| System.Diagnostics.TraceSource | <xref:PostSharp.Patterns.Diagnostics.Backends.TraceSource.TraceSourceLoggingBackend> | [PostSharp.Patterns.Diagnostics.Tracing](https://www.nuget.org/packages/PostSharp.Patterns.Diagnostics.Tracing/) |

Alternatively, you can implement a custom backend. See <xref:custom-logging-backend> for details. 


## Step 3. Configure PostSharp Logging at run time

To configure logging:

1. Identify the startup method of your solution. In a console application and an ASP.NET Core application, the startup method is usually named `Program.Main`. In a XAML application, this is the `Startup` event handler. In an ASP.NET application, this is the `Application_Start` method in the `Global.asax` source file. 


2. To the startup project, add a reference to the package containing the implementation of your logging backend, as listed in the table above.


3. Import the `PostSharp.Patterns.Diagnostics` namespace in the startup file. 

    ```csharp
    using PostSharp.Patterns.Diagnostics;
    ```


4. Add the following code at the top of the startup method:

    ```csharp
    LoggingServices.DefaultBackend = new PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleLoggingBackend();
    ```

    In the code snippet above, you can replace <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleLoggingBackend> with any of the logging backends. 



## Step 4. Configure your logging framework

Remember that PostSharp emits records to the logging framework of your choice. Some of these frameworks may need additional configuration. See the documentation of the logging framework for details.

You may also want PostSharp Logging to process the log records that your code already emits. This is documented in the article specific to your logging framework.

See <xref:backends> for details. 


## Step 5. Instrumentation to ASP.NET and HttpClient (optional)

If you use ASP.NET Core, the legacy ASP.NET or `HttpClient`, we recommend that you add logging to these stacks as described in <xref:logging-instrumentation>. 

This step is required in the following scenarios:

* If you have a distributed application (most modern applications are) and want to correlate the logs of a request whose execution spans several applications. See also <xref:distributed-logging>. 

* If you want to configure per-request (or per-transaction) logging as described in the next step.


## Step 6. Configure logging verbosity

By default, the minimum logging level is `Debug`. This level includes all records that the logging aspect emits by default, and the volume of logs can be too overwhelming to be useful.

We recommend that you set up low verbosity (for instance warning) by default and enable high verbosity when needed, possibly for specific requests, or for specific namespaces. It is possible to store the logging verbosity configuration in an XML file, store it in a cloud drive and configure PostSharp Logging to poll the file for changes every few minutes.

For more details, see <xref:log-enabling>. 


## Result

Now that you have added logging to your project, you will get a very detailed log of your program execution, including parameter values and return values. You are able to add, remove, or rename a method or its parameters with the confidence that your log entries will be kept in sync with each of those changes. Adding logging to your codebase and maintaining it becomes a simple task.

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.LogAttribute>

**Other Resources**

<xref:attribute-multicasting>

