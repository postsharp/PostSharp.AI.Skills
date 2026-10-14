---
uid: log-properties
redirect_from:
  - logging/log-properties
title: "Adding Properties to Messages and Activities"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to add properties to messages and activities in PostSharp. It covers defining properties using an anonymous type, defining inherited properties, and advanced scenarios with logging properties."
---
# Adding Properties to Messages and Activities

When you write a message, open an activity, or close an activity, you can specify additional properties. Properties are name-value pairs that are passed to the logging backend. Unlike message parameters, properties are not rendered into the message text by default. When defined on activities, properties can be inherited by child contexts and activities.


## Defining properties with an anonymous type

Every method that allows you to write a message, open an activity, or close an activity accepts an optional parameter named `options`. The type of this parameter has a constructor that takes a parameter of type `object` called `data`. You can set properties on the message or activity by passing an anonymous object to this constructor. 

For instance, the <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.Write``1(``0@,PostSharp.Patterns.Diagnostics.WriteMessageOptions@)> method accepts an `options` parameter of type <xref:PostSharp.Patterns.Diagnostics.WriteMessageOptions>, and this type has a constructor that accepts an `object`. 

[!code-csharp[](../../code/samples/Logging/LogProperties.cs#anonymous)]

The code above adds two properties to the message: `User` and `Day`. 

You can pass any .NET type to the `data` parameter of the options constructor. By default, all public instance properties will be exported as logging properties unless they throw an exception. 


## Defining inherited properties

By default, all properties defined on an activity apply to the opening and closing message of that activity only. However, it is sometimes useful to include a property in all messages under this activity and under its child activities. This can be done using inherited properties.

You can define an inherited property by annotating it with the <xref:PostSharp.Patterns.Diagnostics.Custom.LoggingPropertyOptionsAttribute> custom attribute and setting its <xref:PostSharp.Patterns.Diagnostics.Custom.LoggingPropertyOptionsAttribute.IsInherited> property to `true`. Of course, you cannot do that with an anonymous type, so you will have to define your own. 

You can also use other properties of <xref:PostSharp.Patterns.Diagnostics.Custom.LoggingPropertyOptionsAttribute>: <xref:PostSharp.Patterns.Diagnostics.Custom.LoggingPropertyOptionsAttribute.IsIgnored>, <xref:PostSharp.Patterns.Diagnostics.Custom.LoggingPropertyOptionsAttribute.IsRendered> and <xref:PostSharp.Patterns.Diagnostics.Custom.LoggingPropertyOptionsAttribute.IsBaggage>. 

For instance, the following class exports a single property to PostSharp Logging, and this property will be inherited by all children messages and activities:

[!code-csharp[](../../code/samples/Logging/LogProperties.cs#attributes)]


## Advanced scenarios with logging properties

When you define properties using a normal .NET object, the object that you pass is wrapped in a <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventData>. This type has two properties: <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventData.Data> and <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventData.Metadata>. The second property, of type <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata>, is responsible for mapping the value of the <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventData.Data> property to logging properties. You can customize this mapping by providing your own implementation of the <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata> class. 

This design, which separates data from metadata, allows you to pass several properties without even allocating a single object on the heap. For instance, you can pass an existing `HttpContext` to PostSharp Logging and use a custom <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata> to decide how this `HttpContext` should be presented to logging backends. 

The following example shows how to expose the HTTP context to properties (this is an abbreviated version of what is implemented by <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetCore.AspNetCoreLogging>): 

[!code-csharp[](../../code/samples/Logging/LogPropertiesMetadata.cs#example)]

You will also create a custom <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata> if you want to define custom transactions. See <xref:custom-logging-transactions> for details. 


