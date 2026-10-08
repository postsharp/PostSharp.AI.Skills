---
uid: custom-formatter
title: "Implementing a Custom Formatter"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to implement a custom formatter in PostSharp for two contexts: logging and caching. It details when to use a custom formatter, how to implement the IFormattable interface, and how to register the custom formatter."
---
# Implementing a Custom Formatter

Formatters are responsible for representing an object as a `string`. Formatters are used in two contexts: logging and caching. This article describes how to implement a custom formatter. 


## When to implement a custom formatter

You may consider implementing a custom formatter in two situations:

* You want the object to be formatted differently in different contexts, that is, you want the logging representation to be different from the caching representation or than the `ToString` representation. 

* The formatting is performance-critical. Since custom formatters are based on the <xref:PostSharp.Patterns.Formatters.UnsafeStringBuilder> class, they are much faster than formatters based on `ToString` or `string.Format`. 


## Implementing the IFormattable interface

If you own the source code of a type, the easiest way to implement a custom formatter is to make the type implement the <xref:PostSharp.Patterns.Formatters.IFormattable> interface, which has a single method named <xref:PostSharp.Patterns.Formatters.IFormattable.Format(PostSharp.Patterns.Formatters.UnsafeStringBuilder,PostSharp.Patterns.Formatters.FormattingRole)>. 

The following example shows how to implement the <xref:PostSharp.Patterns.Formatters.IFormattable> interface: 

```csharp
using PostSharp.Patterns.Diagnostics;
using PostSharp.Patterns.Formatters;

namespace PostSharp.Samples.Logging
{
    class CustomerData : PostSharp.Patterns.Formatters.IFormattable
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        
        [Log(AttributeExclude=true)]
        void Patterns.Formatters.IFormattable.Format(UnsafeStringBuilder stringBuilder, FormattingRole role)
        {
            stringBuilder.Append("{CustomerData FirstName=\"");
            stringBuilder.Append(this.FirstName);
            stringBuilder.Append("\", LastName=\"");
            stringBuilder.Append(this.LastName);
            stringBuilder.Append("\"}");
        }
    }
}
```

> [!TIP]
> To prevent the formatter from being logged, add `[Log(AttributeExclude=true)]` to the formatting method. 

When you implement the <xref:PostSharp.Patterns.Formatters.IFormattable> interface, you do not need to register the formatter because the formatter is the object itself. 


## Implementing the Formatter class

If you do not own the source code of a type, you cannot implement the <xref:PostSharp.Patterns.Formatters.IFormattable>. Instead, you can create a new class derived from the <xref:PostSharp.Patterns.Formatters.Formatter`1> class and implement the <xref:PostSharp.Patterns.Formatters.Formatter`1.Write(PostSharp.Patterns.Formatters.UnsafeStringBuilder,`0)> method. 

The following example illustrates a formatter for the <xref:System.Int32> type. 

```csharp
using PostSharp.Patterns.Diagnostics;
using PostSharp.Patterns.Formatters;

[Log(AttributeExclude=true)]
class FancyIntFormatter : Formatter<int>
{
    public override void Write(UnsafeStringBuilder stringBuilder, int value)
    {
        switch ( value )
        {
            case 0:
                stringBuilder.Append("zero");
                break;

            case 1:
                stringBuilder.Append("one");
                break;

            case 2:
                stringBuilder.Append("two");
                break;

            case 3:
                stringBuilder.Append("three");
                break;

            default:
                stringBuilder.Append(value);
                break;
        }
    }
}
```

> [!TIP]
> To prevent the formatter from being logged, add `[Log(AttributeExclude=true)]` to the formatting method. 


## Registering the custom formatter

Creating a new formatter class does not cause PostSharp to use it. You still need to register it.

Use the following code to register your formatter with PostSharp Logging:

```csharp
LoggingServices.Formatters.Register(new FancyIntFormatter());
```

To use the same formatter in PostSharp Caching, use:

```csharp
CachingServices.Formatters.Register(new FancyIntFormatter());
```

## See Also

**Other Resources**

[Example project: PostSharp.Samples.Logging.Customization](https://samples.postsharp.net/f/PostSharp.Samples.Logging.Customization/)

