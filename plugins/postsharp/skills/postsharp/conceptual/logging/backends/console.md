---
uid: logging-console
redirect_from:
  - logging/logging-console
title: "Logging with System.Console"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to configure PostSharp Logging to output to the console and how to customize console themes for log messages."
---
# Logging with System.Console

This article shows how to target the output of PostSharp Logging to the console and how to configure it.


## Configuring PostSharp Logging to use the console


To send the output of PostSharp Logging to the console:

1. Add PostSharp Logging to your codebase as described in <xref:add-logging>. 


2. In the application startup file, include the following namespace imports:

    [!code-csharp[](../../../code/samples/Logging/LoggingConsole.cs#usings)]

    In the application startup method, include the following code:

    [!code-csharp[](../../../code/samples/Logging/LoggingConsole.cs#configure)]

    The log records then appear in the console:

    [!code-text[](../../../code/samples/Logging/LoggingConsole.txt)]



## Theming the console


### Selecting a console theme

Console themes define the colors used when rendering the log messages. The <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleThemes> class exposes a few themes. The default theme is <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleThemes.Classic>, which copies the colors used by MSBuild. If you do not want any color, use <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleThemes.None>. 

A nicer and more sophisticated theme is <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleThemes.Dark>. 

You can change the current theme by setting the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleLoggingBackendOptions.Theme> as demonstrated in the following example: 

[!code-csharp[](../../../code/samples/Logging/LoggingConsoleThemes.cs#dark)]


### Customizing a theme

The <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme> class provides a convenient implementation of the low-level <xref:PostSharp.Patterns.Diagnostics.Backends.Console.IConsoleTheme> interface. The dark theme is an instance of the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme>. 

The <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme> class simplifies the programming of a theme by allowing only two colors for each log level: a normal color used for the message template, and a highlighted color used for variable parts of the message (typically parameter values). See the methods and properties of the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme> class for details. 

The easiest way to customize the dark theme is to take a copy of the theme by calling the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme.Clone> method and then configure the new object. 

For instance, the following code changes the colors of critical error messages:

[!code-csharp[](../../../code/samples/Logging/LoggingConsoleThemes.cs#customized)]

When the simplifications of the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme> class are too restrictive for your needs, you can directly implement the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.IConsoleTheme> or override the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme.GetStyle(System.String,PostSharp.Patterns.Diagnostics.LogLevel,PostSharp.Patterns.Diagnostics.LogRecordKind,PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleThemeItem)> method of the <xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleTheme> class. 

The following code example displays the message time in a special color:

[!code-csharp[](../../../code/samples/Logging/LoggingConsoleThemes.cs#custom-theme)]

To include the message time, you need the following code:

[!code-csharp[](../../../code/samples/Logging/LoggingConsoleThemes.cs#timestamp)]

## See Also

**Reference**

<xref:PostSharp.Patterns.Diagnostics.Backends.Console.ConsoleLoggingBackend>

**Other Resources**

[Example project: PostSharp.Samples.Logging.Console](https://samples.postsharp.net/f/PostSharp.Samples.Logging.Console/)

