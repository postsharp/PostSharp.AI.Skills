---
uid: custom-contracts
redirect_from:
  - model/contracts/custom-contracts
title: "Creating Custom Contracts"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a step-by-step guide on how to create custom contracts in PostSharp, specifically a contract that throws an exception if a numeric parameter is zero."
---
# Creating Custom Contracts

Given the benefits that contracts provide over manually checking values and throwing exceptions in code, you will likely want to implement your own contracts to perform your own custom checks and handle your own custom types.

The following steps show how to implement a contract that throws an exception if a numeric parameter is zero:


## Implementing a contract that throws an exception if a numeric parameter is zero

1. Use the following namespaces: <xref:PostSharp.Aspects>, <xref:PostSharp.Patterns.Contracts> and <xref:PostSharp.Reflection>. 


2. Derive a class from <xref:PostSharp.Patterns.Contracts.LocationContractAttribute>. 


3. Implement the <xref:PostSharp.Aspects.ILocationValidationAspect> interface in the new contract class that exposes the <xref:PostSharp.Aspects.ILocationValidationAspect`1.ValidateValue(`0,System.String,PostSharp.Reflection.LocationKind,PostSharp.Aspects.LocationValidationContext)> method. Note that this interface must be implemented for each type that is to be handled by the contract. In this example, the contract will handle both `int` and `uint`, so the interface is implemented for both integer types. If additional integer types were to be handled by this class (for example, `long`), then additional implementations of <xref:PostSharp.Aspects.ILocationValidationAspect> would have to be added: 

    [!code-csharp[](../../code/samples/Model/Contracts/NonZeroContract.cs#contract)]

    The <xref:PostSharp.Aspects.ILocationValidationAspect`1.ValidateValue(`0,System.String,PostSharp.Reflection.LocationKind,PostSharp.Aspects.LocationValidationContext)> method takes the value to test, the name of the parameter, property or field, and the usage (that is, whether it is a parameter, property, or field). The method must return an exception if a check fails, or `null` if no exception is to be raised. PostSharp throws the returned exception. 


With the contract created, you can use it. For example, the following methods, which calculate the modulus between two numbers, can use the contract defined above to ensure that neither of their input parameters is zero:

[!code-csharp[](../../code/samples/Model/Contracts/NonZeroContract.cs#usage)]

A program that calls `Mod( 10, 5 )`, `Mod( 10u, 3u )` and `Mod( 10, 0 )` prints the following output:

[!code-text[](../../code/samples/Model/Contracts/NonZeroContract.txt)]


