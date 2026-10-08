---
uid: api
title: "API Reference"
product: "postsharp"
categories: "PostSharp;AOP;PostSharp API"
summary: "The document is an API reference for PostSharp, a design pattern automation tool for Microsoft .NET, detailing its various namespaces and their functions."
---
# API Reference

PostSharp is a design pattern automation tool for Microsoft .NET.

## Namespaces

| Namespace | Description |
|-----------|-------------|
| <xref:PostSharp> | This is the root namespace of PostSharp. |
| <xref:PostSharp.Aspects> | This namespace provides the primitives of the PostSharp aspect-oriented framework, such as abstract aspect classes, aspect interfaces, and advice argument classes. |
| <xref:PostSharp.Aspects.Advices> | This namespace provides custom attributes that allow developing aspects with advice methods, advice parameters and pointcuts. |
| <xref:PostSharp.Aspects.Configuration> | This namespace contains classes and custom attributes configuring the aspects of the PostSharp.Aspects namespace. |
| <xref:PostSharp.Aspects.Dependencies> | This namespace contains types and custom attributes that allow you to define dependencies between aspects and advices, so that the aspect weaver behaves deterministically even if the same element of code is the target of several aspects provided by multiple vendors. |
| <xref:PostSharp.Aspects.Serialization> | This namespace contains types taking care of the process of serializing aspects at build time and deserializing them at run time. |
| <xref:PostSharp.Extensibility> | This namespace defines the semantics of the attribute multicasting mechanism and exposes other types that allow user code to interact with the PostSharp platform. |
| <xref:PostSharp.Reflection> | This namespace extends System.Reflection with new types to reflect the code model. |
| <xref:PostSharp.Collections> | This namespace contains collection types used by the `PostSharp` library. |
| <xref:PostSharp.Constraints> | This namespace allows you to validate your code against predefined or custom design rules. |
| <xref:PostSharp.Serialization> | This namespace implements a portable serializer. |
| <xref:PostSharp.Patterns.Contracts> | This namespace contains ready-made contracts that validate fields, properties and parameters at run time. |
| <xref:PostSharp.Patterns.Diagnostics> | This namespace contains a logging aspect. |
| <xref:PostSharp.Patterns.Model> | This namespace contains an implementation of the Observer, Aggregatable and Disposable patterns. |
| <xref:PostSharp.Patterns.Threading> | This namespace contains an implementation of several threading models, and other thread dispatching aspects. |
| <xref:PostSharp.Patterns.Collections> | This namespace defines collection classes that work with the Aggregatable pattern. |
| <xref:PostSharp.Patterns.Recording> | This namespace implements the undo/redo feature (deprecated). |
| <xref:PostSharp.Patterns.Recording.Operations> | This namespace defines recordable operations (deprecated). |
| <xref:PostSharp.Extensibility.BuildTimeLogging> | This namespace allows you to emit build-time log records. |
| <xref:PostSharp.Patterns> | This is the root namespace for all ready-made implementations of patterns. |
| <xref:PostSharp.Patterns.Caching> | This namespace contains an API to cache method return values as a result of their arguments. |
| <xref:PostSharp.Patterns.Caching.Backends> | This namespace contains implementations of adapters for specific caching backends. |
| <xref:PostSharp.Patterns.Caching.Backends.Azure> | This namespace contains cache invalidators based on Microsoft Azure Service Bus. |
| <xref:PostSharp.Patterns.Caching.Backends.Redis> | This namespace contains an implementation of PostSharp.Patterns.Caching for Redis. |
| <xref:PostSharp.Patterns.Caching.Dependencies> | This namespace defines types that represent caching dependencies. |
| <xref:PostSharp.Patterns.Caching.Formatters> | This namespace contains the formatting role used to format cache keys. |
| <xref:PostSharp.Patterns.Caching.Implementation> | This namespace contains types that are useful when implementing an adapter for a caching backend, but not when consuming the caching API. |
| <xref:PostSharp.Patterns.Caching.Locking> | This namespace implements the feature that can prevent concurrent execution of a cached method. |
| <xref:PostSharp.Patterns.Caching.Serializers> | This namespace contains several serializers, whose role is to serialize cached objects into an array of bytes. |
| <xref:PostSharp.Patterns.Caching.ValueAdapters> | This namespace contains the abstractions that allow you to cache types like Stream or IEnumerable, which could not be cached without adaptation. |
| <xref:PostSharp.Patterns.Collections.Advices> | This namespace defines the abstractions to define advices that can be dynamically added to advisable collections of the PostSharp.Patterns.Collections namespace. |
| <xref:PostSharp.Patterns.Diagnostics.Audit> | This namespace contains the abstractions for the audit feature, which is a special case of logging. |
| <xref:PostSharp.Patterns.Diagnostics.Backends> | This namespace is the base for all adapters for backend logging frameworks. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.ApplicationInsights> | This namespace contains the implementation of the Microsoft Application Insights logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Audit> | This namespace contains the implementation of the audit logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.CommonLogging> | This namespace contains the implementation of the Common.Logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Console> | This namespace contains the implementation of the system console logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.EventSource> | This namespace contains the implementation of the System.Diagnostics.Tracing.EventSource logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Log4Net> | This namespace contains the implementation of the log4net logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Microsoft> | This namespace contains the implementation of the Microsoft.Extensions.Logging logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.NLog> | This namespace contains the implementation of the NLog logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Null> | This namespace contains the implementation of the null logging backend (which has no effect). |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Serilog> | This namespace contains the implementation of the Serilog logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Trace> | This namespace contains the implementation of the System.Diagnostics.Trace logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.TraceSource> | This namespace contains the implementation of the System.Diagnostics.TraceSource logging backend. |
| <xref:PostSharp.Patterns.Diagnostics.Contexts> | This namespace implements different kinds of logging contexts. |
| <xref:PostSharp.Patterns.Diagnostics.Custom> | This namespace contains the implementation of the custom logging front-end API. It is normally not necessary to reference this namespace in user code, as the use of the `var` keyword is recommended. |
| <xref:PostSharp.Patterns.Diagnostics.Custom.Messages> | This namespace contains the implementation of different message types used by the custom logging front-end API. It is normally not necessary to reference this namespace in user code, as the use of the `var` keyword is recommended. |
| <xref:PostSharp.Patterns.Diagnostics.Formatters> | This namespace contains the implementation of formatters used in the context of logging. |
| <xref:PostSharp.Patterns.Diagnostics.RecordBuilders> | This namespace contains the implementations of record builders (a concept similar to a StringWriter). |
| <xref:PostSharp.Patterns.Diagnostics.ThreadingInstrumentation> | This namespace contains aspects that instrument the System.Threading namespace. |
| <xref:PostSharp.Patterns.DynamicAdvising> | This namespace defines the abstractions for dynamically advisable classes, that is, classes of objects into which behaviors can be injected at run time. |
| <xref:PostSharp.Patterns.Formatters> | This namespace contains the implementation of formatters for values of different types. |
| <xref:PostSharp.Patterns.Model.Controls> | This namespace contains XAML controls for the undo/redo feature (deprecated). |
| <xref:PostSharp.Patterns.Model.TypeAdapters> | This namespace defines abstractions that allow you to use the Aggregatable pattern with third-party types. |
| <xref:PostSharp.Patterns.Threading.Models> | This namespace contains the implementation of different threading models. |
| <xref:PostSharp.Patterns.Utilities> | This namespace contains unsorted types. |
| <xref:PostSharp.Patterns.Xaml> | This namespace contains the implementation of XAML-specific patterns. |
| <xref:PostSharp.Reflection.MethodBody> | This namespace defines a code model that allows you to represent a method body as a forest of expression trees. |
| <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetCore> | This namespace allows you to instrument incoming ASP.NET Core requests using PostSharp Logging. |
| <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetFramework> | This namespace allows you to instrument incoming HTTP requests with a legacy IIS-based ASP.NET application and PostSharp Logging. |
| <xref:PostSharp.Patterns.Diagnostics.Adapters.DiagnosticSource> | This namespace allows you to collect logs emitted by DiagnosticSource into PostSharp Logging. |
| <xref:PostSharp.Patterns.Diagnostics.Adapters.HttpClient> | This namespace allows you to instrument outgoing HTTP requests using PostSharp Logging. |
| <xref:PostSharp.Patterns.Diagnostics.Backends.Multiplexer> | This namespace allows you to multiplex the output of PostSharp Logging to several backends instead of just one. |
| <xref:PostSharp.Patterns.Diagnostics.Transactions> | This namespace defines the abstractions for top-level logging transactions such as an incoming request. |
| <xref:PostSharp.Patterns.Diagnostics.Transactions.Model> | This namespace specifies the transaction configuration XML file and the classes and functions available to expressions. |

