---
uid: ignoring-warnings
redirect_from:
  - messaging/ignoring-warnings
title: "Ignoring and Escalating Warnings"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to ignore or escalate warnings in PostSharp, either globally or locally. It provides instructions for adjusting these settings in Visual Studio and MSBuild."
---
# Ignoring and Escalating Warnings

As with conventional compilers, warnings emitted by PostSharp, as well as those emitted by custom code running at build time in PostSharp, can be ignored (in that case they will not be displayed) or escalated into errors.

Warnings can be ignored either globally, using a project-wide setting, or locally for a given element of code. Warnings can be escalated only globally.


## Ignoring or escalating warnings globally

There are several ways to ignore or escalate a warning for a complete project:

* In Visual Studio, in the **PostSharp** tab of the project properties dialog. See <xref:configuration> for details. 

* By defining the `PostSharpDisabledMessages` or `PostSharpEscalatedMessages` MSBuild properties. See <xref:configuration> and <xref:configuration-msbuild> for details. 

> [!NOTE]
> The value `*` can be used to escalate all warnings into errors. 


## Treating warnings as errors with the standard MSBuild properties

PostSharp also reads the standard MSBuild properties that control the warnings of the C# compiler:

* When the `TreatWarningsAsErrors` property is `True`, PostSharp escalates all its warnings, and all the warnings that aspects emit at build time, into errors. The only exception is the warning <xref:ps0131?text=PS0131>, which reports that the project contains no aspect.

  To keep the warnings of PostSharp as warnings in a project that treats the warnings of the compiler as errors, set the `PostSharpTreatWarningsAsErrors` property to `False`.

* The PostSharp message codes that are listed in the `WarningsAsErrors` property are escalated into errors, including <xref:ps0131?text=PS0131>.

* The PostSharp message codes that are listed in the `WarningsNotAsErrors` property are excluded from the escalation caused by `TreatWarningsAsErrors` or `PostSharpTreatWarningsAsErrors`. They are still escalated when they are listed explicitly in `WarningsAsErrors` or `PostSharpEscalatedMessages`.

> [!IMPORTANT]
> Before PostSharp 2027.0, `TreatWarningsAsErrors` and `WarningsAsErrors` had no effect on the warnings of PostSharp. A project that sets `TreatWarningsAsErrors` now fails to build when PostSharp or an aspect emits a warning.


## Ignoring warnings locally

Most warnings are related to a specific element of code. To disable a specific warning for a specific element of code, add the <xref:PostSharp.Extensibility.SuppressWarningAttribute> custom attribute to that element of code, or to any enclosing element of code (for instance, adding the attribute to a type will make it effective for all members of this type). 

You can create your own custom attribute derived from <xref:PostSharp.Extensibility.SuppressWarningAttribute> and make it conditional on a compilation symbol by using the <xref:System.Diagnostics.ConditionalAttribute> custom attribute. 


