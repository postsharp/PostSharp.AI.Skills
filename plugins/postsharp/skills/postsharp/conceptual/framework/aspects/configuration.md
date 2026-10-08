---
uid: aspect-configuration
redirect_from:
  - custompatterns/aspects/advanced/aspect-configuration
title: "Aspect Configuration"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to configure aspects in PostSharp, either declaratively using custom attributes, or imperatively by overriding configuration methods. It also discusses the benefits and limitations of each method. "
---
# Aspect Configuration

Configuration settings of aspects determine how they should be processed by their weaver. Configuration settings are always evaluated at build time. Most aspects have one or more of them. For instance, the aspect type <xref:PostSharp.Aspects.OnExceptionAspect> has a configuration setting determining the type of exceptions handled with this aspect. 

There are two ways to configure an aspect: declarative and imperative.


## Declarative Configuration

You can configure an aspect declaratively by applying the appropriate custom attribute on the aspect class. Aspect configuration attributes are in the namespace <xref:PostSharp.Aspects.Configuration>. Every aspect type has its corresponding type of configuration attribute. The name of the custom attribute starts with the name of the aspect and has the suffix `ConfigurationAttribute`. For instance, the configuration attribute of the aspect class <xref:PostSharp.Aspects.OnExceptionAspect> is <xref:PostSharp.Aspects.Configuration.OnExceptionAspectConfigurationAttribute>. 

Declarative configuration always has precedence over imperative configuration: if a property of the configuration custom attribute is set on the aspect class, or on any parent, the value provided by the corresponding imperative semantic is ignored.

When the configuration attribute sets the same property on a parent class and on a child class, the value set on the child class wins.

These rules apply to each property separately. If no configuration custom attribute sets a property, imperative configuration can still set it.


## Imperative Configuration

A second way to configure an aspect class is to override its configuration methods or set its configuration property.

### Benefits of Imperative Configuration

The advantage of imperative configuration is that it can be arbitrarily complex (since the code of the configuration method is executed inside the weaver). Specifically, it allows the configuration to depend on how the aspect is actually used. For instance, the configuration can depend on the value of a property of the aspect custom attribute.


### Implementation Note

Under the hood, aspects implement the method <xref:PostSharp.Aspects.IAspectBuildSemantics.GetAspectConfiguration(System.Object)>. This method should return a configuration object, derived from the class <xref:PostSharp.Aspects.Configuration.AspectConfiguration>. Every aspect class has its own aspect configuration class. For instance, the configuration class of the aspect class <xref:PostSharp.Aspects.OnExceptionAspect> is <xref:PostSharp.Aspects.Configuration.OnExceptionAspectConfiguration>. The aspect type <xref:PostSharp.Aspects.OnExceptionAspect> implements <xref:PostSharp.Aspects.IAspectBuildSemantics.GetAspectConfiguration(System.Object)> by creating an instance of <xref:PostSharp.Aspects.Configuration.OnExceptionAspectConfiguration>, then it invokes the method <xref:PostSharp.Aspects.OnExceptionAspect.GetExceptionType(System.Reflection.MethodBase)> and copies the return value of this method to the property <xref:PostSharp.Aspects.Configuration.OnExceptionAspectConfiguration.ExceptionType>. Therefore, there are two ways to configure an aspect: either by overriding configuration methods and setting configuration properties (these methods and properties are provided by the framework for convenience only), or by implementing the method <xref:PostSharp.Aspects.IAspectBuildSemantics.GetAspectConfiguration(System.Object)>. If your aspect does not derive from the aspect class <xref:PostSharp.Aspects.OnExceptionAspect>, but directly implements the aspect interface <xref:PostSharp.Aspects.IOnExceptionAspect>, you can use only the latter method. 


