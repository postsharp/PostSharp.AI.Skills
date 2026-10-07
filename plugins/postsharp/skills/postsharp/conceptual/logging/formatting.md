---
uid: logging-customizing
redirect_from:
  - logging/logging-customizing
title: "Formatting Log Records"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "This document explains how to customize log records in PostSharp. It covers different scenarios and procedures, editing a build-time configuration, editing run-time options, implementing a custom formatter, and overriding a backend."
---
# Formatting Log Records

PostSharp offers several ways to customize the appearance and the content of log records. This article maps different customization scenarios to a procedure, then explains each procedure in detail.


## Scenarios

| Scenario | Procedure |
|----------------------------------------------|-----------------------------------------------|
| Include execution time.<br>Emit a warning when execution time exceeds a given threshold.<br>Include source file and line information.<br>Include information about which task or method is being awaited by the `await` operator.  | [Editing a build-time configuration](#editing-a-build-time-configuration) (<xref:PostSharp.Patterns.Diagnostics.LoggingProfile>) in `postsharp.config`.  |
| Include parameter name, type, or value.<br>Include return value.<br>Include `this` parameter. <br>Set the <xref:PostSharp.Patterns.Diagnostics.LogLevel>.  | [Editing a build-time configuration](#editing-a-build-time-configuration) (<xref:PostSharp.Patterns.Diagnostics.LoggingOptions>) in `postsharp.config`.  |
| Set the maximum length of a record.<br>Set formatting options such as the delimiter character or the number of indentation spaces.<br>Include the type name, namespace, or name.<br>Include exception details.<br>Specify which exceptions should be logged. | [Editing run-time options](#editing-run-time-options) (<xref:PostSharp.Patterns.Diagnostics.Backends.TextLoggingBackendOptions> or a derived class).  |
| Change how parameter values of a specific type are rendered. | [Implementing a custom formatter](#implementing-a-custom-formatter).  |
| Change any other behavior. | [Overriding a backend](#overriding-a-backend).  |


## Editing a build-time configuration

Some configuration settings affect how PostSharp generates instructions. These settings can have an important effect on run-time execution speed or assembly size. These settings are defined in the `postsharp.config` file. This file can be private to a project or shared among several projects. See <xref:configuration-system> for details about the `postsharp.config` file. 

Because settings in `postsharp.config` affect code generation, you will have to rebuild your project after you modify this file. 

In large applications, you may need to configure logging differently for different areas or layers in your application. For example, exceptions in the service that cleans up old data in the database can be logged with a "Warning" level, while exceptions in the customer-facing web service must be logged with the "Error" level.

PostSharp enables you to organize your logging options using *logging profiles*. You apply a given logging profile by providing its name as an argument to the <xref:PostSharp.Patterns.Diagnostics.LogAttribute> constructor. 

Logging aspects are assigned to a default profile, as shown in the following table:

| Aspect | Default logging profile |
|--------|-------------------------------------------------------------|
| <xref:PostSharp.Patterns.Diagnostics.LogAttribute> | `default` |
| <xref:PostSharp.Patterns.Diagnostics.LogExceptionAttribute> | `exceptions` |




### Editing the default logging profile

1. Open the file `postsharp.config` in your project. If it does not exist, create an XML file with the following content: 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
    </Project>
    ```

    > [!NOTE]
    > If you create `postsharp.config` in a directory, the file will be used for any project located under that directory. This allows you to share the logging profile among several projects. See <xref:configuration-system> for details. 


2. Edit `postsharp.config` as in the following example: 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
      <Logging xmlns="clr-namespace:PostSharp.Patterns.Diagnostics;assembly:PostSharp.Patterns.Diagnostics">
        <Profiles>
          <LoggingProfile Name="default" IncludeSourceLineInfo="True">
            <DefaultOptions>
              <LoggingOptions IncludeParameterType="True"/>
            </DefaultOptions>
          </LoggingProfile>
        </Profiles>
      </Logging>
    </Project>
    ```

    See the documentation of the <xref:PostSharp.Patterns.Diagnostics.LoggingProfile> and <xref:PostSharp.Patterns.Diagnostics.LoggingOptions> classes for details. 



### Creating and using a new logging profile

1. Configure a logging profile in `postsharp.config`, but choose a different name from `default`, `exceptions` or `audit`. 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
      <Logging xmlns="clr-namespace:PostSharp.Patterns.Diagnostics;assembly:PostSharp.Patterns.Diagnostics">
        <Profiles>
          <LoggingProfile Name="detailed" IncludeSourceLineInfo="True" IncludeExecutionTime="True" IncludeAwaitedTask="True">
            <DefaultOptions>
              <LoggingOptions IncludeParameterType="True" IncludeThisValue="True"/>
            </DefaultOptions>
          </LoggingProfile>
        </Profiles>
      </Logging>
    </Project>
    ```


2. Specify the profile name when you add the <xref:PostSharp.Patterns.Diagnostics.LogAttribute> aspect to a method, type, or project. 

    ```csharp
    [assembly: Log("detailed", AttributeTargetTypes = "My.Namespace.*", AttributePriority = 1, AttributeTargetMemberAttributes = MulticastAttributes.Public)]
    ```

    > [!NOTE]
    > Alternatively, you can create a new aspect class and derive it from <xref:PostSharp.Patterns.Diagnostics.LogAttributeBase>. 



## Editing run-time options

The run-time options are available on the <xref:PostSharp.Patterns.Diagnostics.LoggingBackend.Options> property of the backend class. Each backend can expose a specific set of options. 

Unlike the build-time options, the run-time options are not organized into profiles. They affect all profiles that use the backend you configure.

To edit run-time options, go to the startup method where PostSharp Logging is initialized, and set the backend options:

```csharp
var backend = new ConsoleLoggingBackend();
backend.Options.Delimiter = " \u00A6 ";
backend.Options.Theme = ConsoleThemes.None;
LoggingServices.DefaultBackend = backend;
```


## Implementing a custom formatter

When you include parameter values in log records, PostSharp will invoke the `ToString` method of the object by default. To learn when and how to override the default formatter, see <xref:custom-formatter>. 


## Overriding a backend

If none of the previous options are sufficient, you can create your own backend. Instead of creating your own implementation from scratch, you can derive from any existing implementation and override relevant methods.

See <xref:custom-logging-backend> for details. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.LogAttribute>

**Other Resources**

<xref:attribute-multicasting>
<br><xref:configuration-system>

