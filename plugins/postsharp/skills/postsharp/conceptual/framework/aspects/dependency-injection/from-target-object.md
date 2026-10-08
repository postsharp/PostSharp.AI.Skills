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

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.Composition;
using System.ComponentModel.Composition.Hosting;
using System.ComponentModel.Composition.Primitives;
using System.ComponentModel.Design;
using System.Reflection;
using PostSharp.Aspects;
using PostSharp.Aspects.Advices;
using PostSharp.Extensibility;
using PostSharp.Reflection;
using PostSharp.Serialization;

namespace DependencyResolution.InstanceScoped
{
    public interface ILogger
    {
        void Log(string message);
    }


    [PSerializable]
    public class LogAspect : OnMethodBoundaryAspect, IInstanceScopedAspect
    {
        [IntroduceMember(Visibility = Visibility.Family, OverrideAction = MemberOverrideAction.Ignore)]
        [CopyCustomAttributes(typeof (ImportAttribute))]
        [Import(typeof(ILogger))]
        public ILogger Logger { get; set; }

        [ImportMember("Logger", IsRequired = true)] 
        public Property<ILogger> LoggerProperty;

        public override void OnEntry(MethodExecutionArgs args)
        {
            this.LoggerProperty.Get().Log("OnEntry");
        }

        object IInstanceScopedAspect.CreateInstance(AdviceArgs adviceArgs)
        {
            return this.MemberwiseClone();
        }

        void IInstanceScopedAspect.RuntimeInitializeInstance()
        {
        }
    }


    [Export(typeof (MyServiceImpl))]
    internal class MyServiceImpl
    {
        [LogAspect]
        public void LoggedMethod()
        {
            Console.WriteLine("Hello, world.");
        }
    }

    internal class Program
    {
        private static void Main(string[] args)
        {
            AssemblyCatalog catalog = new AssemblyCatalog(typeof (Program).Assembly);
            CompositionContainer container = new CompositionContainer(catalog);
            MyServiceImpl service = container.GetExport<MyServiceImpl>().Value;
            service.LoggedMethod();
        }
    }


    [Export(typeof (ILogger))]
    internal class ConsoleLogger : ILogger
    {
        public void Log(string message)
        {
            Console.WriteLine(message);
        }
    }
}
```

The following code snippet shows how the logging aspect can be tested:

```csharp
using System;
using System.ComponentModel.Composition;
using System.ComponentModel.Composition.Hosting;
using System.Text;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace DependencyResolution.InstanceScoped.Test
{
    [TestClass]
    public class TestLogAspect
    {
        [TestMethod]
        public void TestMethod()
        {
            TypeCatalog catalog = new TypeCatalog(typeof (TestLogger), typeof (TestImpl));
            CompositionContainer container = new CompositionContainer(catalog);
            TestImpl service = container.GetExport<TestImpl>().Value;
            TestLogger.Clear();
            service.TargetMethod();
            Assert.AreEqual("OnEntry" + Environment.NewLine, TestLogger.GetLog());
        }

        [Export(typeof (TestImpl))]
        private class TestImpl
        {
            [LogAspect]
            public void TargetMethod()
            {
            }
        }
    }

    [Export(typeof (ILogger))]
    internal class TestLogger : ILogger
    {
        public static readonly StringBuilder stringBuilder = new StringBuilder();

        public void Log(string message)
        {
            stringBuilder.AppendLine(message);
        }

        public static string GetLog()
        {
            return stringBuilder.ToString();
        }

        public static void Clear()
        {
            stringBuilder.Clear();
        }
    }
}
```


