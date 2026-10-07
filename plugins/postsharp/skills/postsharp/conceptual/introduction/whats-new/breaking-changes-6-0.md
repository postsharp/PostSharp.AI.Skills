---
uid: breaking-changes-60
redirect_from:
  - introduction/whatsnew/breaking-changes-60
title: "Breaking Changes in PostSharp 6.0"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 6.0 introduces several breaking changes including changes in assembly loading algorithms, MSBuild properties, API removals, and licensing changes. It also discontinues support for certain platforms and features."
---
# Breaking Changes in PostSharp 6.0

PostSharp 6.0 contains the following breaking changes:


## Changes causing silent changes of behavior

* The internal details of the assembly loading algorithms have been changed; therefore, in some rare situations, PostSharp may fail to find or load assemblies in cases that used to be successful with PostSharp 5.0. Please report these cases to the PostSharp support team for individual resolution.

* All MSBuild properties that used to be prefixed `PostSharp30` are now prefixed just `PostSharp`. Check your projects for occurrences of the `PostSharp30` substring. 

* The <xref:PostSharp.Patterns.Contracts.RequiredAttribute> code contract now throws <xref:System.ArgumentOutOfRangeException> instead of <xref:System.ArgumentNullException> when a target element is assigned an empty or white-space string. 


## Changes causing build errors

* Parts of the API that were marked `[Obsolete]` in PostSharp 5.0 have been removed. 

* The way exception handlers and filters are represented in the <xref:PostSharp.Reflection.MethodBody> has changed. They used to be tree nodes, but are now just properties of <xref:PostSharp.Reflection.MethodBody.IBlockExpression>. The <xref:PostSharp.Reflection.MethodBody.MethodBodyVisitor> class has been modified accordingly. 

* The `PostSharp.Patterns.Diagnostics.RecordBuilders.ParameterDirection` has been renamed <xref:PostSharp.Reflection.ParameterKind> and moved to *PostSharp.dll*. 

* The notorious `QueryInterface(object)` extension method has been replaced by the non-extension <xref:PostSharp.Patterns.DynamicAdvising.DynamicAdvisingServices.QueryInterface``1(System.Object,System.Boolean)>, in an effort to limit the pollution of Intellisense suggestions and documentation. 


## Deprecated platforms and features

* Visual Studio 2012 is no longer supported.

* .NET Core SDK 1.1 is no longer supported as a build platform. You can still build .NET Core 1.1 libraries under the .NET Core SDK 2.0 or later. See <xref:requirements> for details. 

* .NET Framework 4.7.1 or later is now required on build platforms. Older versions are still supported as run-time platforms. See <xref:requirements> for details. 

* Windows 7 SP1, Windows 8.1, Windows Server 2008 R2 and early versions of Windows 10 are no longer supported as build platforms. See <xref:requirements> for details. 

* Invoking PostSharp from the command line is no longer supported.

* IncrediBuild is no longer supported.


## Licensing changes

* It is now mandatory to add your license key to the build server or the source repository if you are using the <xref:PostSharp.Patterns.Diagnostics> namespace. See [build-server](xref:logging-license) for details. 


