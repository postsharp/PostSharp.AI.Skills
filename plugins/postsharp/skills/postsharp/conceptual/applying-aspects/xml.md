---
uid: xml-multicasting
redirect_from:
  - custompatterns/aspects/applying/xml-multicasting
title: "Adding Aspects Using XML"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp allows aspects to be added in code and via XML. Adding aspects through XML does not require modifying the source code and can be applied to multiple projects. This is done by specifying an attribute in the project's postsharp.config file."
---
# Adding Aspects Using XML

PostSharp not only allows aspects to be applied in code, but also through XML. This is accomplished by adding them to your project's *postsharp.config* file. See <xref:configuration-system> for information on these files. 

Adding aspects through XML gives the advantage of applying aspects without modifying the source code, which can be useful in some legacy projects. It allows you to apply aspects to several or all projects in your solution without manually including a source file in each project.


## Specifying an attribute in XML

[!code-csharp[](../../code/samples/CustomPatterns/Aspects/Applying/XmlMulticasting/XmlMulticasting.cs#aspect)]

Normally `AutoDataContractAttribute` would be applied to `Customer` in code as follows: 

[!code-csharp[](../../code/samples/CustomPatterns/Aspects/Applying/XmlMulticastingInCode.cs#customer)]

Using XML instead, you can remove the custom attribute from source code and instead specify a `Multicast` element in the PostSharp project file, which is named *postsharp.config* and placed in the same directory as the project file: 

[!code-xml[](../../code/samples/CustomPatterns/Aspects/Applying/XmlMulticasting/postsharp.config)]

In this snippet, the `xmlns:my` attribute associates a prefix with an XML namespace, which must be mapped to the .NET namespace and assembly where the custom attribute classes are defined. In this example, the aspect is defined in the namespace `MyCustomAttributes` of the assembly `XmlMulticasting`: 

```xml
<Multicast xmlns:my="clr-namespace:MyCustomAttributes;assembly:XmlMulticasting">
```

The next line then specifies the custom attribute to apply and the elements to which to apply it:

```xml
<my:AutoDataContractAttribute AttributeTargetTypes="MyNamespace.Customer" />
```

The XML element name must be the name of a class inside the .NET namespace and assembly as defined by the XML namespace. Attributes of this XML element map to public properties or fields of this class.

When the `Customer` class has no custom attribute in source code, but the *postsharp.config* file above is in the project directory, a program that reads the custom attributes of `Customer` by reflection prints:

[!code-text[](../../code/samples/CustomPatterns/Aspects/Applying/XmlMulticasting/XmlMulticasting.txt)]

Note that any property inherited from <xref:PostSharp.Extensibility.MulticastAttribute> can be used here to apply the aspect to several classes at a time. See the section <xref:attribute-multicasting> for details about these properties. 

## See Also

**Other Resources**

<xref:attribute-multicasting>
<br><xref:configuration-system>
