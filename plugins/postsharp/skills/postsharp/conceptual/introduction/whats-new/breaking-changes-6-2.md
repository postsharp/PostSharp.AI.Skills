---
uid: breaking-changes-62
redirect_from:
  - introduction/whatsnew/breaking-changes-62
title: "Breaking Changes in PostSharp 6.2"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document details breaking changes in PostSharp 6.2, including updates to supported platforms and packages, specifically changes in dependencies and .NET Framework requirements for the PostSharp.Patterns.Caching.Redis package."
---
# Breaking Changes in PostSharp 6.2

PostSharp 6.2 contains the following breaking changes:


## Changes in supported platforms and packages

* The *PostSharp.Patterns.Caching.Redis* package now depends on *StackExchange.Redis* v2.0.495 instead of the deprecated *StackExchange.Redis.StrongName* v1.2.3. 

* The *PostSharp.Patterns.Caching.Redis* package now requires .NET Framework 4.6.1 instead of 4.5. 


