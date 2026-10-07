---
uid: simple-tests
redirect_from:
  - custompatterns/aspects/testing/simple-tests
title: "Testing Run-Time Logic"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to design a test strategy for aspects in PostSharp. It emphasizes the need for large test coverage and provides an example of testing a caching aspect."
---
# Testing Run-Time Logic

When designing a test strategy for aspects, it is fundamental to understand that aspects cannot be used in isolation. They are always used in the context of the code artifact to which they are applied. Therefore, when writing an aspect, two kinds of test artifacts must be written:

* *Test target code* to which the aspect will be applied. 

* *Test invocation code* that invokes the target code and verifies that the combination of the aspect and the target code exhibits the intended behavior. 


## Achieving large test coverage

As with other code, you have to test the aspect with input context that varies enough to produce high code coverage.

In the case of aspects, the input context is composed of the following items:

* *Arguments of the aspect itself*, that is, constructor arguments and property values. If the aspect behavior depends on aspect arguments, high code coverage of the aspect requires varying aspect arguments. 

* *Target code* can be considered part of the input arguments of the aspect. For instance, if an aspect contains logic that depends on the method being static or non-static, you should test the aspect against both static and non-static methods. 

* *Arguments of the target code* can affect the run-time behavior of the aspect. For instance, a buggy aspect may incorrectly handle null arguments. 


## Example: testing a caching aspect

The following example demonstrates how to test a caching aspect. High code coverage is achieved by varying the target code and testing with null and non-null parameters.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Samples
{
    [TestClass]
    public class TestCacheAspect
    {
        private static int invocations;

        // Instance method without parameters

        [TestMethod]
        public void TestInstanceMethodWithoutParameter()
        {
            int call1 = this.InstanceMethodWithoutParameter();
            int call2 = this.InstanceMethodWithoutParameter();

            Assert.AreEqual(call1, call2);
        }

        [Cache]
        private int InstanceMethodWithoutParameter()
        {
            return invocations++;
        }

        // Static method without parameters

        [TestMethod]
        public void TestStaticMethodWithoutParameter()
        {
            int call1 = StaticMethodWithoutParameter();
            int call2 = StaticMethodWithoutParameter();

            Assert.AreEqual(call1, call2);
        }

        [Cache]
        private static int StaticMethodWithoutParameter()
        {
            return invocations++;
        }

        //  Instance method with parameters

        [TestMethod]
        public void TestInstanceMethodWithParameter()
        {
            int call1a = this.InstanceMethodWithParameter("foo");
            int call2a = this.InstanceMethodWithParameter(null);
            int call1b = this.InstanceMethodWithParameter("foo");
            int call2b = this.InstanceMethodWithParameter(null);

            Assert.AreEqual(call1a, call1b);
            Assert.AreEqual(call2a, call2b);
            Assert.AreNotEqual(call1a, call2a);
        }

        [Cache]
        private int InstanceMethodWithParameter(string param)
        {
            return invocations++;
        }


        [TestMethod]
        public void TestInstanceTaskMethodWithParameters()
        {
            int invocationsOld = invocations;

            int call11a = this.InstanceTaskMethodWithParameters(1, 1).Result;
            int call11b = this.InstanceTaskMethodWithParameters(1, 1).Result;

            Assert.AreEqual(call11a, call11b);
            Assert.AreEqual(1, invocations - invocationsOld);
        }

        #region InstanceTaskMethodWithParameters

        [Cache]
        private Task<int> InstanceTaskMethodWithParameters(int a, int b)
        {
            return Task.Run(() =>
            {
                invocations++;
                return a + b;
            });
        }

        #endregion
    }
}
```


