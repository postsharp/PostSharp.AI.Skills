---
uid: breaking-changes-50
redirect_from:
  - introduction/whatsnew/breaking-changes-50
title: "Breaking Changes in PostSharp 5.0"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 5.0 introduces breaking changes including silent changes of behavior, build errors, build warnings, deprecated platforms, and licensing changes. Users need to modify their code accordingly."
---
# Breaking Changes in PostSharp 5.0

PostSharp 5.0 contains the following breaking changes:


## Changes causing silent changes of behavior

* The <xref:PostSharp.Aspects.OnMethodBoundaryAspect> aspect will now advise the state machine of async methods and iterators by default instead of the kick-off method. The `ApplyToStateMachine` property is deprecated in favor of <xref:PostSharp.Aspects.OnMethodBoundaryAspect.SemanticallyAdvisedMethodKinds> and its default value is now `true`. The behavior of existing code that ignored the `PS0215` warning will silently change. 


## Changes causing build errors

* The `PostSharp.Reflection.Syntax` namespace has been renamed `PostSharp.Reflection.MethodBody` and classes inside this namespace have been renamed to reflect the fact this namespace is definitely not a syntax tree. (We were tired of apologizing for this misnaming.) You must modify your code (hopefully just a few files) otherwise it will no longer build. 

* The `PostSharp.MessageLocation` class has been moved to the `PostSharp.Extensibility` namespace. You must modify your code (hopefully just a few files) otherwise it will no longer build. 

* The `PostSharp.Patterns.Diagnostics` has been completely revamped. You must modify your code (hopefully just a few files) otherwise it will no longer build. 

* The `GenericArgs` family of classes, a legacy from PostSharp 1.5 that no longer worked, was deleted. 

* It is no longer allowed to add an aspect or an advice to an anonymous method or to a method of a compiler-generated type, including the `MoveNext` method of a state machine type. To avoid the error, implementations of <xref:PostSharp.Aspects.IAspectProvider> and pointcuts should use `method.GetSemanticInfo().IsSelectable` to determine whether a method is a valid aspect or advice target. 


## Changes causing build warnings

* The `PostSharp.IgnoreWarningAttribute` class has been renamed <xref:PostSharp.Extensibility.SuppressWarningAttribute> and moved to the `PostSharp.Extensibility` namespace. You will get an obsolescence warning until you modify your code. 


## Deprecated platforms

* Silverlight, Windows Phone and WinRT are no longer supported. These projects will no longer build.

* Xamarin is temporarily unsupported. These projects will no longer build. Please contact our support team if you are affected by this change.


## Licensing changes

* PostSharp Professional has been renamed PostSharp Framework and no longer includes PostSharp Logging. Please contact our sales team if you have a PostSharp Professional license and rely on PostSharp Logging.

* PostSharp Express has been renamed PostSharp Essentials and is no longer compatible with PostSharp Express 4.2 and earlier. The licensing of PostSharp Express was modified in PostSharp 4.3, but PostSharp 4.3 included a backward-compatibility licensing mode. It has now been disabled.

* Use of the license server is now restricted to the new PostSharp Enterprise license. Please contact our sales team if you are affected by this change.

> [!TIP]
> Please contact our sales team if you are a commercial user and are affected by the licensing changes. We will find a solution that is acceptable for both.


