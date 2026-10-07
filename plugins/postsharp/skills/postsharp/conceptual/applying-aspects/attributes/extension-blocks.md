---
uid: extension-blocks-multicast
redirect_from:
  - custompatterns/aspects/applying/extension-blocks-multicast
title: "C# 14 Extension Blocks and Multicasting"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how C# 14 extension blocks are handled by PostSharp's multicast engine in PostSharp 2026.0, including the AllowExtensionBlockMembers property and ReflectionHelper.IsExtensionBlockMetadata method."
---
# C# 14 Extension Blocks and Multicasting
## Overview

Starting with PostSharp 2026.0, the multicast engine provides support for C# 14 extension blocks. By default, aspects and multicast attributes are not applied to extension block members to prevent unexpected behavior. You can explicitly enable this using the <xref:PostSharp.Extensibility.MulticastAttribute.AllowExtensionBlockMembers> property.

## How extension blocks are implemented in IL

The C# compiler implements extension block members as static methods (including properties) and a set of special metadata types. These metadata types are intended for the C# compiler to match extension methods and properties with the receiver type.

At the IL level and when viewed through `System.Reflection`, the compiler generates:

- Static implementation methods (e.g., `ExtensionMethod` and `get_ExtensionProperty`)
- Nested compiler-generated metadata types that describe the extension block structure

For example, consider this C# 14 extension block:

```csharp
public static class TestClassExtensions
{
    extension<TInstance>(TInstance instance)
    {
        public TInstance ExtensionMethod() => instance;

        public TInstance ExtensionProperty => instance;
    }
}
```

The compiler generates nested types similar to:

```csharp
public static class TestClassExtensions
{
    // Compiler-generated nested metadata types representing the extension block.
    [SpecialName]
    private static class <G>$BA41CFE2B5EDAEB8C1B9062F59ED4D69<TInstance>
    {
        // Pure metadata classes.
        [SpecialName]
    	public static class <M>$DE9F57644BDC66EDA3F1FD365749DA9F;

        [SpecialName]
    	public static class <M>$7A2C8F3E91B4D5A6C0E9F1B2D4A7C8E3;
    	

        // Original extension members without implementations.
        [ExtensionMarker("<M>$DE9F57644BDC66EDA3F1FD365749DA9F")]
        public TInstance ExtensionMethod() => throw null;

        [ExtensionMarker("<M>$DE9F57644BDC66EDA3F1FD365749DA9F")]
        public TInstance ExtensionProperty => throw null;
    }

    // Static implementation methods with received parameter.
    public static TInstance ExtensionMethod<TInstance>(TInstance instance) => instance;

    public static TInstance get_ExtensionProperty<TInstance>(TInstance instance) => instance;
}
```

Since both the implementation methods and the metadata types may not be expected by existing aspects, the multicasting algorithm in PostSharp 2026.0 and later behaves as follows:

- **Extension block metadata types are always skipped** - These compiler-generated nested types and their members are never eligible for aspect application
- **Extension block implementation methods require opt-in** - Static methods that implement extension members are skipped by default but can be targeted by setting `AllowExtensionBlockMembers = true`

## Enabling extension block support

To apply aspects to extension block members, do one of the following:

- Add the aspect directly to the extension block member.
- Set the <xref:PostSharp.Extensibility.MulticastAttribute.AllowExtensionBlockMembers> property to `true` when multicasting the aspect from an outer scope, such as the declaring type.
- Add <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> to the aspect type and set <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowExtensionBlockMembers> to `true`.

### Example: Applying aspects directly to extension members

When applied explicitly to extension members, aspects are added without the need to change any of the properties.

```csharp
[PSerializable]
class LoggingAspect : OnMethodBoundaryAspect
{
    public override void OnEntry(MethodExecutionArgs args)
    {
        Console.WriteLine($"Entering: {args.Method.Name}");
    }
}

public static class TestClassExtensions
{
    extension<TInstance>(TInstance instance)
    {
        [LoggingAspect]
        public TInstance ExtensionMethod() => instance;

        [LoggingAspect]
        public TInstance ExtensionProperty
        {
            get => instance;
            set { }
        }
    }
}
```

### Example: Multicasting with AllowExtensionBlockMembers set to true

After setting <xref:PostSharp.Extensibility.MulticastAttribute.AllowExtensionBlockMembers> to `true` and multicasting on the whole class, members within extension blocks will be eligible for the aspect.

```csharp
[PSerializable]
class LoggingAspect : OnMethodBoundaryAspect
{
    public override void OnEntry(MethodExecutionArgs args)
    {
        Console.WriteLine($"Entering: {args.Method.Name}");
    }
}

[LoggingAspect(AllowExtensionBlockMembers = true)]
public static class TestClassExtensions
{
    extension<TInstance>(TInstance instance)
    {
        public TInstance ExtensionMethod() => instance;

        public TInstance ExtensionProperty
        {
            get => instance;
            set { }
        }
    }
}
```

### Example: Changing default behavior for custom aspects

Setting the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowExtensionBlockMembers> property to `true` on the aspect class will make extension members automatically eligible for the aspect:

```csharp
[MulticastAttributeUsage(MulticastTargets.Method, AllowExtensionBlockMembers = true)]
[PSerializable]
public class LoggingAspect : OnMethodBoundaryAspect
{
    public override void OnEntry(MethodExecutionArgs args)
    {
        Console.WriteLine($"Entering: {args.Method.Name}");
    }
}

[LoggingAspect]
public static class TestClassExtensions
{
    extension<TInstance>(TInstance instance)
    {
        public TInstance ExtensionMethod() => instance;

        public TInstance ExtensionProperty
        {
            get => instance;
            set { }
        }
    }
}
```

## IAspectProvider and IAdviceProvider

C# 14 extension blocks present a challenge when using <xref:PostSharp.Aspects.IAspectProvider> or <xref:PostSharp.Aspects.Advices.IAdviceProvider> because the `System.Reflection` API exposes the compiler-generated IL artifacts rather than the source-level C# syntax.

When implementing `IAspectProvider` or `IAdviceProvider`, you receive reflection objects (`Type`, `MethodInfo`, etc.) that represent the IL structure. For extension blocks, this means you will encounter:

- Static implementation methods for extension members
- Compiler-generated nested metadata types (e.g., `<G>$BA41CFE2B5EDAEB8C1B9062F59ED4D69<TInstance>`)

Attempting to target extension block metadata types or their members will result in error LA0167.

To make your `IAspectProvider` or `IAdviceProvider` safe to use with extension blocks, remember that:

- **Extension member implementation methods are safe to target** - These are the static methods that implement the actual extension logic and can have aspects applied to them.
- **Extension block metadata types must be avoided** - Targeting these compiler-generated types or their members will result in error LA0167.

PostSharp 2026.0 introduces the <xref:PostSharp.Reflection.ReflectionHelper.IsExtensionBlockMetadata> method to help identify and filter extension block metadata artifacts. This method allows you to distinguish between regular types/members and compiler-generated extension block metadata.

### Example: Filtering extension block metadata

When implementing `IAspectProvider`, use `IsExtensionBlockMetadata` to skip compiler-generated metadata types:

```csharp
using PostSharp.Aspects;
using PostSharp.Reflection;
using PostSharp.Serialization;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;

[PSerializable]
public class MyAspectProvider : TypeLevelAspect, IAspectProvider
{
    public IEnumerable<AspectInstance> ProvideAspects(object targetElement)
    {
        Type type = (Type)targetElement;

        // Get all nested types, but filter out extension block metadata
        var nestedTypes = type.GetNestedTypes(BindingFlags.Public | BindingFlags.NonPublic)
            .Where(t => !ReflectionHelper.IsExtensionBlockMetadata(t));

        foreach (var nestedType in nestedTypes)
        {
            // Safely process non-metadata nested types
            yield return new AspectInstance(nestedType, new MyAspect());
        }

        // Extension member implementation methods are safe to target
        var methods = type.GetMethods(BindingFlags.Public | BindingFlags.Static)
            .Where(m => !ReflectionHelper.IsExtensionBlockMetadata(m));

        foreach (var method in methods)
        {
            yield return new AspectInstance(method, new MyMethodAspect());
        }
    }
}
```



## Backward compatibility

Classic extension methods using the `this` modifier continue to work as before and are not affected by the `AllowExtensionBlockMembers` property. They are always eligible for aspect application according to existing multicast rules.


## See Also

**Reference**

<xref:PostSharp.Extensibility.MulticastAttribute.AllowExtensionBlockMembers>
<br><xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowExtensionBlockMembers>
<br><xref:PostSharp.Reflection.ReflectionHelper.IsExtensionBlockMetadata>

**Other Resources**

<xref:multicast-conceptual>
<br><xref:attribute-multicasting>
<br><xref:whats-new-20260>
<br><xref:iaspectprovider>
<br>[C# 14 Extension Members - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/extension)
<br>[Exploring extension members - .NET Blog](https://devblogs.microsoft.com/dotnet/csharp-exploring-extension-members/)
