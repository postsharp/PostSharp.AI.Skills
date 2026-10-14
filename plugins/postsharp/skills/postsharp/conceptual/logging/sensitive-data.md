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

PostSharp Logging can capture all parameters and return values. It hides some parameters automatically, as described below. Any other sensitive data passed as a parameter or returned by a method is included in the log.

### Parameters hidden automatically

PostSharp Logging writes `*******` instead of the value of the following parameters:

* parameters whose name contains `password` or `pwd`, ignoring case, for example `pwd`, `oldPassword` or `PasswordHash`;
* parameters of type <xref:System.Security.SecureString>.

Return values are never hidden automatically, and neither are parameters with other names, such as `secret` or `token`.

[!code-csharp[](../../code/samples/Root/HidingSensitiveDataByName.cs#example)]

These methods produce the following log:

[!code-text[](../../code/samples/Root/HidingSensitiveDataByName.txt)]

There are two techniques to prevent other confidential data from being leaked into a log:

### Option 1. Use a wrapper object for all sensitive parameters and return values

The first option is storing sensitive data in a wrapper object that hides its value from the log by implementing `ToString`, the <xref:PostSharp.Patterns.Formatters.IFormattable> interface, or both. For more convenient debugging, you can use the <xref:System.Diagnostics.DebuggerDisplayAttribute> custom attribute.

Here is an example implementation:

[!code-csharp[](../../code/samples/Root/HidingSensitiveDataWrapper.cs#example)]

When a logged method returns or receives a `Sensitive<string>`, the log does not contain the value:

[!code-text[](../../code/samples/Root/HidingSensitiveDataWrapper.txt)]

The benefit of using this pattern is that it will force you to change all APIs to pass sensitive values transitively, making you less likely to forget some occurrences.

### Option 2. Mark parameters and return values with [NotLogged]

Alternatively, mark every sensitive parameter and return value with the <xref:PostSharp.Patterns.Diagnostics.NotLoggedAttribute?text=[NotLogged]> custom attribute. It causes PostSharp Logging to use `*******` instead of the actual value. You can add this attribute to the return value using `[return: NotLogged]`.

[!code-csharp[](../../code/samples/Root/HidingSensitiveDataNotLogged.cs#example)]

These methods produce the following log:

[!code-text[](../../code/samples/Root/HidingSensitiveDataNotLogged.txt)]

## Sensitive data scanning

The problem with the above approaches is that you can easily forget to exclude a few sensitive occurrences. It is possible to increase the confidence level by deliberate testing of logging. However, it is impossible to be entirely sure there will be no leak.

The general idea is to deliberately use well-known passwords or sensitive data, such as `P@ssw0rd` or `4242424242424242`, to run your application through all possible execution paths, and to check that there is no occurrence of your well-known password in the output log.

While executing this test, log verbosity must be set to its maximum level for all classes and methods.

