---
uid: express-limitations
redirect_from:
  - deploymentconfiguration/licensing/express-limitations
title: "PostSharp Essentials"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp Essentials is a free edition of PostSharp with unlimited free features and premium features limited to 1,000 lines of code. Premium features include semantic advising of async methods, instrumenting explicit properties, production mode logging, and integration with higher levels of PostSharp SDK."
---
# PostSharp Essentials

PostSharp Essentials is a free edition of PostSharp that includes both free and premium features. Free features are available in unlimited amount. However, premium features can be applied only to 1,000 lines of code.

Note that any edition of PostSharp includes the free entitlements of PostSharp Essentials in addition to the premium features covered by this edition.


## Free features

The following features of PostSharp are free and unlimited in PostSharp Essentials:


### Method decorators and interceptors

Both aspects <xref:PostSharp.Aspects.OnMethodBoundaryAspect> and <xref:PostSharp.Aspects.MethodInterceptionAspect> can be used for free, but semantic advising of async methods is a premium feature. 

To disable semantic advising, you have to set the <xref:PostSharp.Aspects.OnMethodBoundaryAspect.SemanticallyAdvisedMethodKinds> or <xref:PostSharp.Aspects.MethodInterceptionAspect.SemanticallyAdvisedMethodKinds> property to `None` in the aspect constructor. 

> [!CAUTION]
> Make sure you understand the consequences of disabling semantic advising in your use cases. See <xref:semantic-advising> for details. 


### INotifyPropertyChanged

The <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> can be used for free, but only for auto-implemented properties. Instrumenting explicit properties is a premium feature. 

Explicit properties are instrumented by default. To limit PostSharp to instrument only auto-implemented properties, you can set the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute.ExcludeExplicitProperties> property to `true`. 


### Contracts

All contracts can be used for free without limitation. See <xref:contracts> for details. 


### PostSharp Logging developer mode

You can apply logging to any code size using PostSharp Essentials, but you need to explicitly enable the developer mode. Otherwise, production mode will be used, which is considered a premium feature.

To enable developer mode, set the `LoggingDeveloperMode` property to `true` in *postsharp.config*. See <xref:logging-license> for details. 


### PostSharp SDK

You can create your own add-ins, or use community add-ins, that use the lower layers of PostSharp SDK. Integrating with higher levels (Aspect Weaver) is a premium feature covered by PostSharp Framework or PostSharp Ultimate. This premium feature is not provided at all by PostSharp Essentials.


## Enforcement of the solution-level limit

You can find more information about how we count lines of code in <xref:licensing-counting-lines>. 

The limitation of 1,000 enhanced lines of code per solution is implemented not by looking at the *sln* file, but by counting the lines of code of classes in all assemblies that are referenced by the current assembly. That is, the limit is actually 1,000 lines of code of enhanced types in the whole assembly closure. The same limit also applies to each project.


## Diagnosing licensing issues

If you do not understand why PostSharp requires a commercial license, you can generate a licensing diagnostic log by building your project with the following command line:

```text
msbuild /t:Rebuild /v:detailed /p:PostSharpTrace=Licensing > msbuild.log
```

See [Capturing licensing diagnostic logs](xref:license-troubleshooting#capturing-licensing-diagnostic-logs) for details.


