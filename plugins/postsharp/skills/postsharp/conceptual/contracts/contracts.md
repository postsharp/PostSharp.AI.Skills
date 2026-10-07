---
uid: contracts
redirect_from:
  - model/contracts
title: "Contracts"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses PostSharp Code Contracts, which improve code readability, are inheritable, and customizable for localization. It also outlines sections on adding, customizing contracts and exceptions."
---
# Contracts

Throwing exceptions upon detecting a bad or unexpected value is a good programming practice called *precondition checking*. However, writing the same checks over and over in different areas of the code base is tedious, error-prone, and difficult to maintain. 

PostSharp Code Contracts have the following features and benefits:

* More readable. PostSharp Code Contracts are represented as custom attributes, so there is less code to read and understand.

* Inherited. You can add a PostSharp Code Contract attribute to an interface method parameter and it will automatically be enforced in all implementations of this method.

* Localizable. It's easy to display the error message in the user's language, even if you didn't design for this scenario upfront.


## In this chapter

| Section | Description |
|---------|-------------|
| <xref:adding-contracts> | This section demonstrates how to add contracts to code and how inheritance works. |
| <xref:custom-contracts> | This section explains how to create your own contract attributes. |
| <xref:contract-custom-exceptions> | This section explains how to replace the default exception types with your own. |
| <xref:contract-localization> | This section describes how to customize the texts of exceptions that are thrown when a contract is violated. |

## See Also

**Reference**

<xref:PostSharp.Patterns.Contracts.RequiredAttribute>
<br><xref:PostSharp.Patterns.Model>
<br><xref:PostSharp.Patterns.Contracts>
<br><xref:PostSharp.Aspects>
<br><xref:PostSharp.Reflection>
<br><xref:PostSharp.Patterns.Contracts.LocationContractAttribute>
<br><xref:PostSharp.Patterns.Contracts.LocationContractAttribute.ErrorMessage>
<br><xref:PostSharp.Aspects.ILocationValidationAspect>
<br><xref:PostSharp.Aspects.ILocationValidationAspect`1.ValidateValue*>
