---
uid: logging-hiding-sensitive-data
redirect_from:
  - hiding-sensitive-data
title: "Hiding Sensitive Data When Logging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides guidelines for hiding sensitive data when logging with PostSharp, to prevent potential security breaches and maintain user data confidentiality. It suggests using wrapper objects or marking parameters with [NotLogged] to avoid exposing sensitive data."
---

# Hiding Sensitive Data When Logging
When combining PostSharp Logging and multicasting to add logging to large parts of your application automatically, you risk exposing passwords and other sensitive data in logs. Storing sensitive information in logs poses significant risks, as logs can be accessed by unauthorized individuals or inadvertently leaked. Exposing passwords and sensitive data in logs can lead to unauthorized access and other malicious activities. It also causes data privacy and confidentiality issues and may violate regulations like GDPR. By keeping such information hidden, you can prevent potential security breaches and maintain the confidentiality of user data.


## Avoiding the exposure of sensitive data

PostSharp Logging can capture all parameters and return values. If sensitive data is passed as a `string` parameter or return value, it will be included in the log. 

There are two techniques to prevent confidential data from being leaked into a log:

### Option 1. Use a wrapper object for all sensitive parameters and return values

The first option is storing sensitive data in a wrapper object that hides its value from the log by implementing `ToString`, the <xref:PostSharp.Patterns.Formatters.IFormattable> interface, or both. For more convenient debugging, you can use the <xref:System.Diagnostics.DebuggerDisplayAttribute> custom attribute.

Here is an example implementation:

```csharp
[Log(AttributeExclude=true)]
[DebuggerDisplay("{Value}")]
public readonly struct Sensitive<T> : PostSharp.Patterns.Formatters.IFormattable
{
    public T Value { get; }

    public Sensitive( T value )
    {
        Value = value;
    }

    public override string? ToString() => "(Confidential)";

    void PostSharp.Patterns.Formatters.IFormattable.Format(UnsafeStringBuilder stringBuilder, FormattingRole role)
    {
        stringBuilder.Append("(Confidential)");
    }
}

```

The benefit of using this pattern is that it will force you to change all APIs to pass sensitive values transitively, making you less likely to forget some occurrences.

### Option 2. Mark parameters and return values with [NotLogged]

Alternatively, mark every sensitive parameter and return value with the <xref:PostSharp.Patterns.Diagnostics.NotLoggedAttribute?text=[NotLogged]> custom attribute. It causes PostSharp Logging to use `*******` instead of the actual value. You can add this attribute to the return value using `[return: NotLogged]`.


```csharp
[Log]
private static void NotLoggedMethod( string p1, [NotLogged] string p2 )
{
}

[Log]
[return: NotLogged]
private static string NotLoggedReturnValueMethod( string p1,  string p2 )
{
    return p1 + p2;
}

[Log]
[return: NotLogged]
private static (string a, string b) NotLoggedReturnValueTupleMethod( string p1, string p2 )
{
    return (p1, p2);
}

```

## Sensitive data scanning

The problem with the above approaches is that you can easily forget to exclude a few sensitive occurrences. It is possible to increase the confidence level by deliberate testing of logging. However, it is impossible to be entirely sure there will be no leak.

The general idea is to deliberately use well-known passwords or sensitive data, such as `P@ssw0rd` or `4242424242424242`, to run your application through all possible execution paths, and to check that there is no occurrence of your well-known password in the output log.

While executing this test, log verbosity must be set to its maximum level for all classes and methods.

