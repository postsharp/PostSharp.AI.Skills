---
uid: adding-contracts
redirect_from:
  - model/contracts/adding-contracts
title: "Adding Contracts to Code"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use PostSharp's Contract attributes to validate parameters, fields, and properties in code, ensuring data integrity and reducing boilerplate code. It also covers contract inheritance in derived classes."
---
# Adding Contracts to Code

This section describes how to add a contract to a field, property, or parameter.


## Introduction

Consider the following method that checks whether a valid string has been passed in:

[!code-csharp[](../../code/samples/Model/Contracts/ContractsManualChecks.cs#example)]

In this example, checks have been added to ensure that both parameters contain a valid string. A better solution is to place the logic which performs this check into its own reusable class, especially when boilerplate logic is involved, and then reuse this class whenever the check needs to be performed.

PostSharp’s Contract attributes move such checks out of the code and into parameter attributes. For example, PostSharp’s <xref:PostSharp.Patterns.Contracts.RequiredAttribute> contract could be used to simplify the example as follows: 

[!code-csharp[](../../code/samples/Model/Contracts/ContractsRequiredParameters.cs#example)]

In this example, the <xref:PostSharp.Patterns.Contracts.RequiredAttribute> attribute performs the check for null, thus eliminating the need to write the boilerplate code for the check in line with other code. A program that calls this method with valid names, and then with `null` as the last name, prints the full name and the message of the <xref:System.ArgumentNullException>:

[!code-text[](../../code/samples/Model/Contracts/ContractsRequiredParameters.txt)]

A contract can also be applied to a property or field as shown in the following example:

[!code-csharp[](../../code/samples/Model/Contracts/ContractsRequiredProperty.cs#example)]

Using a contract in a property ensures that the value being passed into `set` is validated before the logic (if any) for `set` is executed.

Similarly, a contract can be used directly on a field to validate the value being assigned to the field:

[!code-csharp[](../../code/samples/Model/Contracts/ContractsRequiredField.cs#example)]

In this example, `firstName` will be validated by the `Required` contract before being assigned to `mFirstName`. Placing a contract on a field provides the added benefit of validating the field regardless of where it is set from. 

Note that PostSharp also includes a number of built-in contracts that range from checks for null values to testing for valid phone numbers. You can also develop your own contracts with custom logic for your own types as described below.

## Adding contracts


### Adding a contract to an element of code

1. Add the [PostSharp.Patterns.Common](https://www.nuget.org/packages/PostSharp.Patterns.Common/) package to your project. 


2. Import the <xref:PostSharp.Patterns.Contracts> namespace into your file: 

    [!code-csharp[](../../code/samples/Model/Contracts/ContractsReturnValue.cs#using)]


3. Add the contract custom attribute to the parameter, field, property, or return value. See the <xref:PostSharp.Patterns.Contracts> namespace for a list of available contract attributes. For example: 

    [!code-csharp[](../../code/samples/Model/Contracts/ContractsReturnValue.cs#example)]

    This example makes sure that both the parameters and the return value are not null.



## Contract inheritance

PostSharp ensures that any contracts that have been applied to an abstract, virtual, or interface method are inherited along with that method in derived classes, all without the need to re-specify the contract in the derived methods. This is shown in the following example:

[!code-csharp[](../../code/samples/Model/Contracts/ContractsInheritance.cs#example)]

Here, the `ICustomerModel.SetFullName` method specifies that the `firstName` and `lastName` parameters are required using the <xref:PostSharp.Patterns.Contracts.RequiredAttribute> attribute. Since the `CustomerModel.SetFullName` method implements this method, these attributes will also be applied to its parameters. Calling `CustomerModel.SetFullName` with a `null` first name throws an exception with the following message:

[!code-text[](../../code/samples/Model/Contracts/ContractsInheritance.txt)]

> [!NOTE]
> If the derived class exists in a separate assembly, that assembly must be processed by PostSharp and must reference the `PostSharp.Patterns.Common` package.


