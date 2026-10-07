---
uid: logging-license
redirect_from:
  - logging/logging-license
title: "Licensing of PostSharp Logging: Production vs Developer Mode"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp Logging has two modes: Production Mode for unlimited logging and Developer Mode which stops emitting logs 24 hours after compilation. Production Mode is premium while Developer Mode is free."
---
# Licensing of PostSharp Logging: Production vs Developer Mode


## Production Mode

This is the default mode. It is meant for logging of the application without a time limit. Production Mode is a premium feature and is included in PostSharp Logging and PostSharp Ultimate.


## Developer Mode

Unlike Production Mode, an application built with PostSharp Logging in Developer Mode will stop emitting logging messages 24 hours after it has been compiled. After this period of time, your application will keep working, but the logging messages will no longer be emitted.

PostSharp Logging in Developer Mode is a free feature for projects of any number of lines of code.

To enable Developer Mode, set the `LoggingDeveloperMode` property to `true` in `postsharp.config`. 

You can do this by creating a file named `postsharp.config` in your project directory with the following content: 

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.postsharp.org/1.0/configuration">
    <Property Name="LoggingDeveloperMode" Value="True" />
</Project>
```

See <xref:configuration-system> for details. 


