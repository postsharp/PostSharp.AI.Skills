---
uid: log-custom-activities
redirect_from:
  - logging/log-custom-activities
title: "Working with Custom Activities"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to work with custom activities in PostSharp, including logging an activity, measuring execution time, and changing the default level for exceptions. It also provides examples of code implementation."
---
# Working with Custom Activities

You often need to log the beginning and the end of an activity, for example `Starting to read MyFile.xml` and `Succeeded to read MyFile.xml` or `Cannot read MyFile.xml: unexpected element at line 5`. An *activity* is anything that begins and eventually ends. 

You can open custom activities using the <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.OpenActivity*>, <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.LogActivity*> or <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.LogActivityAsync*> method. 

Activities define a scope, so anything executing within that activity is properly indented. Therefore, activities are hierarchical: they have a notion of a child and of a parent and form a tree.

When you use the <xref:PostSharp.Patterns.Diagnostics.LogAttribute> aspect to log a method, it translates into an activity behind the scenes, which belongs to the same tree as custom activities. 


## Logging an activity, the explicit way


To log a custom activity:

1. Add a <xref:PostSharp.Patterns.Diagnostics.LogSource> to your class as described in <xref:log-custom-messages>. 


2. In a `using` statement, invoke the <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.OpenActivity``1(``0@,PostSharp.Patterns.Diagnostics.OpenActivityOptions@)> method. Specify the activity description with the <xref:PostSharp.Patterns.Diagnostics.FormattedMessageBuilder.Formatted*> or <xref:PostSharp.Patterns.Diagnostics.SemanticMessageBuilder.Semantic*> method. 

    > [!TIP]
    > We strongly advise that you use the `var` keyword to receive the return value of <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.OpenActivity``1(``0@,PostSharp.Patterns.Diagnostics.OpenActivityOptions@)> method because the return type of the method depends on the number and types of arguments sent to <xref:PostSharp.Patterns.Diagnostics.FormattedMessageBuilder.Formatted*> or <xref:PostSharp.Patterns.Diagnostics.SemanticMessageBuilder.Semantic*>. Alternatively, you can use a variable of type <xref:PostSharp.Patterns.Diagnostics.Custom.Messages.IMessage>, but this will cause boxing of the <xref:PostSharp.Patterns.Diagnostics.LogActivity`1> value type. 


3. Wrap the code of the activity with a `try/catch` construct. 


4. In the `catch` block, call <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetException(System.Exception,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)>. 


5. In the `try`, call <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetSuccess(PostSharp.Patterns.Diagnostics.CloseActivityOptions@)> or <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetResult``1(``0,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)> or, if you need more flexibility, <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetOutcome``1(PostSharp.Patterns.Diagnostics.LogLevel,``0@,System.Exception,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)>. 


> [!CAUTION]
> All projects that use the <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.OpenActivity``1(``0@,PostSharp.Patterns.Diagnostics.OpenActivityOptions@)> method in async methods must be processed by PostSharp, otherwise the execution of your application will fail. Therefore your projects must have a reference to the `PostSharp.Patterns.Common` package (a reference to `PostSharp.Patterns.Common.Redist` is not sufficient), and PostSharp must not be disabled on this project. Even if you do not have asynchronous code, it is recommended that you use the `PostSharp.Patterns.Common` package for performance reasons. 


### Example

The following code shows how to open and close an activity.

[!code-csharp[](../../code/samples/Logging/LogCustomActivities.cs#example)]

This code produces the following output:

[!code-text[](../../code/samples/Logging/LogCustomActivities.txt)]


## Logging an activity the compact way, with an anonymous method

If you do not want to write the boilerplate code of the previous example over and over again, you can use the <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.LogActivity*> or <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.LogActivityAsync*> method. These methods will execute a specified delegate within a new custom activity, and will automatically invoke the <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetSuccess(PostSharp.Patterns.Diagnostics.CloseActivityOptions@)>, <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetResult``1(``0,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)> or <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetException(System.Exception,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)> method as needed. 

The inconvenient consequence of using <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.LogActivity*> is that it will likely result in the allocation of an instance of a closure class on the heap, and affect performance even when logging is disabled. 


### Example

The following code does almost the same as the previous example but with fewer lines of code. The main difference is that, in the current example, the <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetResult``1(``0,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)> method is invoked instead of <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetOutcome``1(PostSharp.Patterns.Diagnostics.LogLevel,``0@,System.Exception,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)>, and there is heap allocation even when logging is disabled. 

[!code-csharp[](../../code/samples/Logging/LogCustomActivitiesCompact.cs#example)]

This code produces the following output:

[!code-text[](../../code/samples/Logging/LogCustomActivitiesCompact.txt)]


## Changing the default level for exceptions

When you open an activity with a specific level, say <xref:PostSharp.Patterns.Diagnostics.LogSource.Debug>, and then close the activity successfully, the close message will be emitted with the same level as the open message, i.e. <xref:PostSharp.Patterns.Diagnostics.LogSource.Debug>. 

However, when the activity fails with an exception, the close message level will be <xref:PostSharp.Patterns.Diagnostics.LogSource.Failure>, whose value can be changed only by the <xref:PostSharp.Patterns.Diagnostics.LogSource.WithLevels(PostSharp.Patterns.Diagnostics.LogLevel,PostSharp.Patterns.Diagnostics.LogLevel)> method. See [Configuring the default logging levels centrally](xref:log-custom-messages#configuring-the-default-logging-levels-centrally) for information about changing the default levels. 


## Measuring the execution time of activities

You can measure the execution time of all activities by enabling the <xref:PostSharp.Patterns.Diagnostics.LoggingBackendOptions.IncludeActivityExecutionTime> property: 

[!code-csharp[](../../code/samples/Logging/LogActivityExecutionTime.cs#example)]

This will cause the execution time to be appended to the close message.

## See Also

**Other Resources**

<xref:log-properties>

