---
uid: emitting-errors
redirect_from:
  - messaging/emitting-errors
title: "Emitting Errors, Warnings, and Messages"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to emit custom messages, warnings, and errors using PostSharp's messaging facility. It also advises on using a MessageSource for storing all message texts in a single location."
---
# Emitting Errors, Warnings, and Messages

Custom code running in PostSharp at build time can use the messaging facility to emit its own messages, warnings, and errors. These messages appear in the MSBuild output, in Visual Studio, or in both. User-emitted warnings can be ignored or escalated using the same mechanism as for system messages.


## Emitting messages

If you have only a few messages to emit, you can use one of the overloads of the `Write` method of the <xref:PostSharp.Extensibility.Message> class. 

All messages must have a severity (<xref:PostSharp.Extensibility.SeverityType>), a message number (used as a reference when ignoring or escalating messages), and a message text. Additionally, and preferably, messages may have a location (<xref:PostSharp.Extensibility.MessageLocation>). 

> [!NOTE]
> To benefit from the possibility to ignore messages locally, you should always provide a relevant location with your messages. Previous API overloads, which did not require a message location, are considered obsolete.

> [!TIP]
> Do not use `string.Format` to format your messages. Instead, pass message arguments to the messaging facility, which will format some argument types, for instance reflection objects, in a more readable way. 


## Emitting messages using a message source

If you want the text of all messages to be stored in a single location, you have to emit messages through a <xref:PostSharp.Extensibility.MessageSource>. Typically, you would create a singleton instance of <xref:PostSharp.Extensibility.MessageSource> for each component, and associate each instance with a message dispenser. A message dispenser is a custom-written class implementing the <xref:PostSharp.Extensibility.IMessageDispenser> interface. The <xref:PostSharp.Extensibility.MessageDispenser> provides a convenient abstract implementation. 

> [!NOTE]
> Although it is tempting to use a `ResourceManager` as the backend of a message dispenser, doing so comes with a non-negligible performance penalty because of the cost of instantiating the `ResourceManager`. 


