---
uid: binary-formatter-security
redirect_from:
  - deploymentconfiguration/deployment/binary-formatter-security
title: "BinaryFormatter Security"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The .NET Core 3.1 BinaryFormatter, used for binary serialization, is considered insecure. PostSharp aspects are serialized at build time, but the vulnerability does not apply to PostSharp's use of serialization. The recommended action is to use PortableFormatter for serializing aspects."
---
# BinaryFormatter Security
In .NET Core 3.1, <xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter>, which is used for binary serialization of CLR objects, began to be considered insecure and dangerous. In .NET 5.0, <xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter> started to throw an exception upon its use in ASP.NET Core applications. In .NET 8.0, more serialization-related APIs started to be obsolete and by default, `BinaryFormatter` is disabled (throwing exceptions) in all .NET 8.0 projects with the exception of WinForms and WPF projects. In .NET 9.0, the implementation of the `BinaryFormatter` class and its related types has been removed, and all members of these types now only have exception-throwing implementation.

The attack vector of this vulnerability is deserialization of data that could be manipulated by the attacker, which can result in execution of arbitrary commands under credentials of the process that executed the deserialization.

## Impact of vulnerability on PostSharp 

PostSharp allows multiple ways to serialize aspects. The <xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter> class is used if the type has the `[Serializable]` attribute in C#. After PostSharp 4.0, the preferred method of serialization became through <xref:PostSharp.Serialization.PSerializableAttribute?text=[PSerializable]>, which results in using <xref:PostSharp.Serialization.PortableFormatter>, our own efficient and portable implementation specialized in serializing aspects. 

Since PostSharp 2024.0, using `[Serializable]` on aspect classes will result in a build-time error <xref:la0206?text=LA0206>. 

In legacy applications that require usage of binary serialization, you can disable this error by setting the `PostSharpBinaryFormatterAllowed` MSBuild property to `true`.

When building under .NET 9.0+ in PostSharp 2025.0 and later, using this setting will automatically reference the `System.Runtime.Serialization.Formatters` package as a build-time dependency. Additional steps must be taken to enable run-time support for aspect binary deserialization. See more in the [.NET documentation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/binaryformatter-migration-guide/compatibility-package).

> [!CAUTION]
> Using `PostSharpBinaryFormatterAllowed` is not recommended and is unsupported.

> [!NOTE]
> In releases before PostSharp 2024.0, using `[Serializable]` may result in build-time warning LA0205, which can be suppressed through `NoWarn` MSBuild property.

## Recommended actions

Since the usage of `BinaryFormatter` is unsupported by Microsoft and in most projects using it would cause a run-time exception, it is recommended to use `PortableFormatter` for serializing aspects instead.

All aspects and types that are used for aspect serialization should use `[PSerializable]` instead of `[Serializable]`.

## See Also

**Reference**

<xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter>
<br><xref:PostSharp.Serialization.PSerializableAttribute>

**Other Resources**

<xref:aspect-serialization>
<br>[BinaryFormatter serialization methods are obsolete and prohibited in ASP.NET apps](https://docs.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/5.0/binaryformatter-serialization-obsolete)
<br>[BinaryFormatter security guide](https://docs.microsoft.com/en-us/dotnet/standard/serialization/binaryformatter-security-guide)
