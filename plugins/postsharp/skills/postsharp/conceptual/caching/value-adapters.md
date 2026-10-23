---
uid: caching-value-adapters
redirect_from:
  - caching/caching-value-adapters
title: "Caching Special Types with Value Adapters"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use PostSharp's value adapters to cache special types that cannot be directly cached. It also provides instructions for implementing a custom value adapter."
---
# Caching Special Types with Value Adapters

Some types, for instance the <xref:System.Collections.Generic.IEnumerator`1> interface or the <xref:System.IO.Stream> class, cannot be directly cached because the position of the enumerator or stream can be changed by the caller. Some other interfaces such as <xref:System.Collections.Generic.IEnumerable`1> cannot be cached because the real value may be a LINQ expression, and it is not useful to cache the LINQ expression itself. However, you may still want to cache methods returning these types. If you wrote the code manually, you would cache a different type, for example a list or an array of bytes. 

PostSharp addresses this problem with the concept of a *value adapter*. A value adapter allows you to store a type other than the return type of the cached method. The method return value is called the *exposed value* because this is the value exposed by your API. The exposed value must be type-compatible with the method return type. The value that is actually stored in the cache is called the *stored value*. For instance, for a method returning a <xref:System.IO.Stream>, the stored value is an array of bytes and the exposed value is a <xref:System.IO.MemoryStream>. 


## Standard value adapters

The following value adapters are used automatically by default:

| Return type | Stored type | Exposed type | Comments |
|-------------------------------------------------|-------------|--------------|----------|
| <xref:System.Collections.Generic.IEnumerable`1> | <xref:System.Collections.Generic.List`1> | <xref:System.Collections.Generic.List`1> |  |
| <xref:System.Collections.Generic.IEnumerator`1> | <xref:System.Collections.Generic.List`1> | An internal implementation of <xref:System.Collections.Generic.IEnumerator`1> | The <xref:System.Collections.IEnumerator.Reset> method is not supported by the exposed value.  |
| <xref:System.IO.Stream> | <xref:System.Byte> []  | <xref:System.IO.MemoryStream> |  |


## Implementing a custom value adapter


### Creating and registering a value adapter

1. Add a reference to the [PostSharp.Patterns.Caching](https://www.nuget.org/packages/PostSharp.Patterns.Caching/) package. 


2. Create a class implementing the <xref:PostSharp.Patterns.Caching.ValueAdapters.IValueAdapter`1> interface or the <xref:PostSharp.Patterns.Caching.ValueAdapters.IValueAdapter> interface. The easiest way is to derive it from the <xref:PostSharp.Patterns.Caching.ValueAdapters.ValueAdapter`1> abstract class. 


3. Register the class from the previous step using one of the overloads of the <xref:PostSharp.Patterns.Caching.ValueAdapters.ValueAdapterFactory.Register(System.Type,PostSharp.Patterns.Caching.ValueAdapters.IValueAdapter)> method. 

    Each caching backend has its own instance of the <xref:PostSharp.Patterns.Caching.ValueAdapters.ValueAdapterFactory> class available via the <xref:PostSharp.Patterns.Caching.Implementation.CachingBackend.ValueAdapters> property. 


> [!NOTE]
> Null values are handled automatically outside of the value adapters.


### Example

In this example, we create the ``EnumerableValueAdapter`1`` class, which transforms instances of the <xref:System.Collections.Generic.IEnumerable`1> interface into an array. 

[!code-csharp[](../../code/samples/Caching/Core/CachingValueAdapters.cs#example)]

The output of this sample is:

[!code-text[](../../code/samples/Caching/Core/CachingValueAdapters.txt)]


