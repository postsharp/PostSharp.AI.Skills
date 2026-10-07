---
uid: configuration-serialization
redirect_from:
  - deploymentconfiguration/configuration/configuration-serialization
title: "Including CLR Objects in Configuration"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document describes how PostSharp includes a facility to describe CLR objects using XML, including rules for XML serialized objects and attribute formatting. It also outlines specific rules for the Multicast section of the configuration file."
---
# Including CLR Objects in Configuration

PostSharp includes a basic facility to describe CLR objects using XML. This facility is used to implement the [Multicast](xref:configuration-schema#multicast) and [logging profiles](xref:logging-customizing#editing-a-build-time-configuration) sections of the configuration file, and can be used to define custom sections. 

The facility is deliberately limited. It was designed only to provide the same features as custom attributes in programming languages.


## Basic rules

The following basic rules apply to XML-serialized objects:

* The local name of the XML element must exactly match the type name of the CLR type. An exception to this rule is that the `Attribute` suffix can be omitted. 

* The XML namespace of the element must be in the form `clr-namespace:namespace;assembly:assembly` where *namespace* is the namespace of the CLR type and *assembly* is the name of the assembly declaring the type. 

* The type must have a public parameterless constructor.

* Names of XML attributes must exactly match the name of a public field or property of the CLR type.


## Formatting of attributes

Values of XML attributes, mapping to CLR fields and properties, must be formatted according to the rule relevant for each type:

| Type | Formatting |
|------|------------|
| Intrinsics | An intrinsic is any of the following types: `bool`, `char`, `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `float`, `double` or `string`. Conversion is done using the <xref:System.Convert> class.  |
| Arrays | A semicolon-separated list of elements. |
| Enumerations | A list of enumeration member names separated by the character `\|`. The values of the members are combined as flags. In the `Multicast` section, you can also use `+` as a separator.  |
| Types | An assembly-qualified type name. |
| Object | Fields and properties of type <xref:System.Object> are not supported.  |


## Specific rules for the Multicast section

The following additional rules apply to the [Multicast](xref:configuration-schema#multicast) section of the configuration file: 

* The class must derive from <xref:PostSharp.Extensibility.MulticastAttribute>. 

* By default, the <xref:PostSharp.Extensibility.MulticastAttribute.AttributePriority> property is set to the 1-based position of the XML element in the section. You can set this property explicitly to override this default. 


