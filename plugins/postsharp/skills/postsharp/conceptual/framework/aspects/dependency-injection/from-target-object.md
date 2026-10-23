---
uid: importing-dependencies
redirect_from:
  - custompatterns/aspects/testing/importing-dependencies
title: "Importing Dependencies from the Target Object"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to import dependencies from target objects in PostSharp by implementing the IInstanceScopedAspect interface and using custom attributes. It includes a step-by-step guide and code examples."
---
# Importing Dependencies from the Target Object

Aspects are believed to be difficult to test mainly because they are statically scoped by default, that is, aspect objects are stored in static fields. However, any aspect can be made instance-scoped if it implements the <xref:PostSharp.Aspects.IInstanceScopedAspect> interface. See <xref:aspect-lifetime> for more information about aspect scopes. 

Instance-scoped aspects can consume dependencies from the objects to which they are applied. They can also add dependencies to the target objects.

For instance, an aspect can consume a service `ILogger` using the following procedure.


## Consuming a service from an instance-scoped aspect

1. Add a public property named `Logger` of type `ILogger` to the aspect, and add the <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute> custom attribute. The aspect then adds a property to the target class. Set its `OverrideAction` property to `MemberOverrideAction.Ignore` to ignore the property if it already exists in the target type or if another aspect has added it. 


2. Add the custom attributes <xref:System.ComponentModel.Composition.ImportAttribute> and <xref:PostSharp.Aspects.Advices.CopyCustomAttributesAttribute> to the `Logger` property. The aspect then adds the `[Import]` custom attribute to the `Logger` property that it adds to the target class. 


3. Add a public field named `LoggerProperty` of type `Property<ILogger>` to the aspect class, and add the <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> custom attribute to this field, with `"Logger"` as the parameter value. The aspect can then read the `Logger` property even if it is defined outside the aspect. 


4. The aspect can now consume the dependency by calling `this.LoggerProperty.Get()`. 


The next example illustrates this procedure.


## Example: testable logging aspect that consumes the dependency from the target object

The following code snippet shows a logging aspect and how it could be used in production code:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/ImportingDependencies.cs#production)]

The program prints:

[!code-text[](../../../../code/samples/CustomPatterns/Testing/ImportingDependencies.txt)]

The following code snippet shows how the logging aspect can be tested:

[!code-csharp[](../../../../code/samples/CustomPatterns/Testing/ImportingDependenciesTest.cs#test)]


