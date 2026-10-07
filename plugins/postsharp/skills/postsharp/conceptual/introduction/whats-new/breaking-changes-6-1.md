---
uid: breaking-changes-61
redirect_from:
  - introduction/whatsnew/breaking-changes-61
title: "Breaking Changes in PostSharp 6.1"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 6.1 introduces breaking changes including a change in logging behavior, method signature changes in the API for building custom logging back-ends, and replacement of the Logger class. It also drops support for Visual Studio 2013 and .NET Core SDK 2.0."
---
# Breaking Changes in PostSharp 6.1

PostSharp 6.1 contains the following breaking changes:


## Changes causing silent changes of behavior

* Earlier versions of PostSharp Logging used the *role* as the name of the logger (in most frameworks) or context (in Serilog). PostSharp Logging 6.1 uses the source type instead, which makes more sense, but is a breaking change. 


## Changes causing build errors

* We changed some method signatures in the API that allows you to build customized logging back-ends. The chances that you may be affected are minor.


## Changes causing build warnings

* We replaced the <xref:PostSharp.Patterns.Diagnostics.Logger> class by <xref:PostSharp.Patterns.Diagnostics.LogSource>. The new API requires C# 7.3. The old <xref:PostSharp.Patterns.Diagnostics.Logger> still works. You will get a warning only when you call the <xref:PostSharp.Patterns.Diagnostics.Logger.GetLogger(System.String)> method. For existing developments, we recommend to disable the warning using a `#pragma` directive. 


## Deprecated platforms and features

* Visual Studio 2013 is no longer supported.

* .NET Core SDK 2.0 is no longer supported as a build platform. See <xref:requirements> for details. 


