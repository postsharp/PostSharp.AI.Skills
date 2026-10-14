---
uid: contract-custom-exceptions
redirect_from:
  - model/contracts/contract-custom-exceptions
title: "Customizing Contract Exceptions"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to customize contract exceptions in PostSharp by overriding exception types for built-in contracts, using localization service in custom exception factories, and using the exception factory from custom contracts."
---
# Customizing Contract Exceptions

Although PostSharp Contracts raise standard .NET exceptions like <xref:System.ArgumentNullException>, there are cases where you may want the contracts to raise different types of exceptions. For instance, you may have to work with existing code that expects different exception types. In this scenario, it is possible to override the mechanism by which PostSharp Contracts instantiate exceptions. 


## Overriding exception types for built-in contracts

Built-in contracts use the <xref:PostSharp.Patterns.Contracts.ContractExceptionFactory> class exposed by the <xref:PostSharp.Patterns.Contracts.ContractServices.ExceptionFactory> property to instantiate their exceptions. You can override this property with your own implementation of the <xref:PostSharp.Patterns.Contracts.ContractExceptionFactory> class. 

This class has a single method of interest named <xref:PostSharp.Patterns.Contracts.ContractExceptionFactory.CreateException(PostSharp.Patterns.Contracts.ContractExceptionInfo)>. It should instantiate and return an <xref:System.Exception> based on the instance of the <xref:PostSharp.Patterns.Contracts.ContractExceptionInfo> object passed by the aspect. 


### Creating a custom exception factory

1. Create a class that inherits from <xref:PostSharp.Patterns.Contracts.ContractExceptionFactory>. 


2. Override the <xref:PostSharp.Patterns.Contracts.ContractExceptionFactory.CreateException(PostSharp.Patterns.Contracts.ContractExceptionInfo)> method. For the cases that your exception factory is designed to handle, instantiate and return an <xref:System.Exception>. For the cases that you do not want to handle, call `base.CreateException()` to invoke the next node in the chain of responsibility. 


3. Put an instance of the factory into the <xref:PostSharp.Patterns.Contracts.ContractServices.ExceptionFactory> property on application start. When constructing the new instance, pass the previous value of the <xref:PostSharp.Patterns.Contracts.ContractServices.ExceptionFactory> property. 



### Example

The following example overrides the default behavior of the <xref:PostSharp.Patterns.Contracts.RequiredAttribute>. When it calls the `Oops` method with a `null` argument, this program gets a `CustomException`. 

[!code-csharp[](../../code/samples/Model/Contracts/ContractExceptionFactoryCustom.cs#example)]

The output of this program is:

[!code-text[](../../code/samples/Model/Contracts/ContractExceptionFactoryCustom.txt)]


## Using the localization service in custom exception factories

The default exception factory uses the localization service to render human-readable error messages. You can use the same facility in your own exception factories. It is exposed on the <xref:PostSharp.Patterns.Contracts.ContractServices.LocalizedTextProvider> property.

For details about localization, see <xref:contract-localization>.


### Example

Like the previous example, this example overrides the exception type raised in case of violation of a `[Required]` contract, but it uses the standard contract localization service to format the error message. 

[!code-csharp[](../../code/samples/Model/Contracts/ContractExceptionFactoryLocalized.cs#example)]

With this factory, the program of the previous example prints the standard message:

[!code-text[](../../code/samples/Model/Contracts/ContractExceptionFactoryLocalized.txt)]


## Using the exception factory from custom contracts

If you want other developers to be able to customize the behavior of your custom contracts, it is a good idea to use the same pattern as for built-in contracts, that is, to use both the <xref:PostSharp.Patterns.Contracts.ContractExceptionFactory> and <xref:PostSharp.Patterns.Contracts.ContractLocalizedTextProvider> facilities. 

If your custom contracts throw different exception types or use different error messages than the built-in contracts, you will need to inject your own implementations of these classes in the chain of responsibility. Otherwise, you can use the default implementations.


### Example

The following contract uses the built-in exception factory to create a standard <xref:System.ArgumentOutOfRangeException> when a value is zero. Note that if you want a custom message shown, you need to use a custom `messageId` and then implement localization as described in <xref:contract-localization>. 

[!code-csharp[](../../code/samples/Model/Contracts/NonZeroWithExceptionFactory.cs#example)]

> [!NOTE]
> You need to pass a message ID that is known by the localization service. Otherwise, an exception is thrown. If the message identified by `messageId` needs to format any additional arguments into the error message, put them in the `additionalArguments` array. The message IDs of built-in contracts are defined as public const fields of <xref:PostSharp.Patterns.Contracts.ContractLocalizedTextProvider>. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Contracts.ContractExceptionFactory>
<br><xref:PostSharp.Patterns.Contracts.ContractExceptionInfo>
<br><xref:PostSharp.Patterns.Contracts.ContractLocalizedTextProvider>

**Other Resources**

<xref:contract-localization>
