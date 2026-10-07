---
uid: multicast-override
redirect_from:
  - custompatterns/aspects/applying/multicast-override
title: "Overriding and Removing Aspect Instances"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to override and remove aspect instances in PostSharp. It covers applying multiple instances of the same aspect, manually and automatically overriding an aspect instance, and deleting an aspect instance."
---
# Overriding and Removing Aspect Instances

Having multiple instances of the same aspect on the same element of code is sometimes the desired behavior. With multicasting custom attributes (<xref:PostSharp.Extensibility.MulticastAttribute>), it is easy to end up with that situation. Indeed, many multicasting paths can lead to the same target. 

However, most of the time, a different behavior is preferred. You can define a method-level aspect on the type (this aspect would apply to all methods) and override (or even exclude) the aspect on a specific method.

The multicasting engine has both the ability to apply multiple aspect instances on the same target, and the ability to replace or remove custom attributes.


## Understanding the Multicasting Algorithm

Before going ahead, it is important to understand the multicasting algorithm. The algorithm relies on a notion of *order of processing* of aspect instances. 

> [!IMPORTANT]
> This section covers how PostSharp handles multiple instances of the same aspect type for the sole purpose of computing how aspect instances should be overridden or removed. See <xref:aspect-dependencies> to understand how to cope with multiple instances of different aspects. 

The following rules apply:

* Aspect instances defined on a container (for instance a type) are always processed before instances defined on an item of that container (for instance a method). Elements of code are processed in the following order: assembly, module, type, field, property, event, method, parameter.

* When multiple aspect instances are defined on the same level, they are sorted by increasing value of the <xref:PostSharp.Extensibility.MulticastAttribute.AttributePriority>. 

The algorithm builds a list of aspect instances applied (directly and indirectly) on an element of code, sorts these instances, and processes overrides or removals as described below.


## Applying Multiple Instances of the Same Aspect

The property <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowMultiple> determines whether multiple instances of the same aspect are allowed on an element of code. By default, this property is set to `true` for all aspects. 

In the following example, the methods in type `MyClass` are enhanced by one, two and three instances of the `Trace` aspect (see code comments). 

```csharp
using System;
using System.Diagnostics;
using PostSharp.Aspects;
using PostSharp.Extensibility;
using PostSharp.Serialization;
using Samples3;

[assembly: Trace(AttributeTargetTypes = "Samples3.My*", Category = "A")]
[assembly: Trace(AttributeTargetTypes = "Samples3.My*",
    AttributeTargetMemberAttributes = MulticastAttributes.Public, Category = "B")]

namespace Samples3
{
    [PSerializable]
    public sealed class TraceAttribute : OnMethodBoundaryAspect
    {
        public string Category { get; set; }

        public override void OnEntry(MethodExecutionArgs args)
        {
            Trace.WriteLine("Entering " +
                            args.Method.DeclaringType.FullName + "." + args.Method.Name, this.Category);
        }
    }


    public class MyClass
    {
        // This method will have 1 Trace aspect with Category set to A.
        private void Method1()
        {
        }

        // This method will have 2 Trace aspects with Category set to A, B
        public void Method2()
        {
        }

        // This method will have 3 Trace aspects with Category set to A, B, C.
        [Trace(Category = "C")]
        public void Method3()
        {
        }
    }
}
```


## Overriding an Aspect Instance Manually

You can require an aspect instance to override any previous one by setting the aspect property <xref:PostSharp.Extensibility.MulticastAttribute.AttributeReplace>. This is equivalent to a deletion followed by an insertion (see below). 

In the following example, the first two methods of type `MyClass` are enhanced by aspects applied at assembly level, but these aspects are replaced by a different one on `Method3`. 

```csharp
using System;
using System.Diagnostics;
using PostSharp.Aspects;
using PostSharp.Extensibility;
using PostSharp.Serialization;
using Samples5;

[assembly: Trace(AttributeTargetTypes = "Samples5.My*", Category = "A")]
[assembly: Trace(AttributeTargetTypes = "Samples5.My*",
    AttributeTargetMemberAttributes = MulticastAttributes.Public, Category = "B")]

namespace Samples5
{
    [PSerializable]
    public sealed class TraceAttribute : OnMethodBoundaryAspect
    {
        public string Category { get; set; }

        public override void OnEntry(MethodExecutionArgs args)
        {
            Trace.WriteLine("Entering " +
                            args.Method.DeclaringType.FullName + "." + args.Method.Name, this.Category);
        }
    }


    public class MyClass
    {
        // This method will have 1 Trace aspect with Category set to A.
        private void Method1()
        {
        }

        // This method will have 2 Trace aspects with Category set to A, B.
        public void Method2()
        {
        }

        // This method will have 1 Trace aspect with Category set to C.
        [Trace(Category = "C", AttributeReplace = true)]
        public void Method3()
        {
        }
    }
}
```


## Overriding an Aspect Instance Automatically

To cause a new aspect instance to automatically override any previous one, the aspect developer must disallow multiple instances by annotating the aspect class with the custom attribute <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> and setting the property <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute.AllowMultiple> to `false`. 

In the following example, the methods in type `MyClass` are enhanced by a single `Trace` aspect: 

```csharp
using System;
using System.Diagnostics;
using PostSharp.Aspects;
using PostSharp.Extensibility;
using PostSharp.Serialization;
using Samples4;

[assembly: Trace(AttributeTargetTypes = "Samples4.My*", AttributePriority = 1, Category = "A")]
[assembly: Trace(AttributeTargetTypes = "Samples4.My*",
    AttributeTargetMemberAttributes = MulticastAttributes.Public, AttributePriority = 2, Category = "B")]

namespace Samples4
{
    [MulticastAttributeUsage(MulticastTargets.Method, AllowMultiple = false)]
    [PSerializable]
    public sealed class TraceAttribute : OnMethodBoundaryAspect
    {
        public string Category { get; set; }

        public override void OnEntry(MethodExecutionArgs args)
        {
            Trace.WriteLine("Entering " +
                            args.Method.DeclaringType.FullName + "." + args.Method.Name, this.Category);
        }
    }


    public class MyClass
    {
        // This method will have 1 Trace aspect with Category set to A.
        private void Method1()
        {
        }

        // This method will have 1 Trace aspect with Category set to B.
        public void Method2()
        {
        }

        // This method will have 1 Trace aspect with Category set to C.
        [Trace(Category = "C")]
        public void Method3()
        {
        }
    }
}
```


## Deleting an Aspect Instance

The <xref:PostSharp.Extensibility.MulticastAttribute.AttributeExclude> property removes any previous instance of the same aspect on a target. 

This is useful, for instance, when you need to exclude a target from the matching set of a wildcard expression. For example:

```csharp
[assembly: Configurable( AttributeTargetTypes = "BusinessLayer.*" )]

          namespace BusinessLayer
          {
            [Configurable( AttributeExclude = true )]
            public static class Helpers
            {

            }
          }
```


