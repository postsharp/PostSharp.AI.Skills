---
uid: breaking-changes-64
redirect_from:
  - introduction/whatsnew/breaking-changes-64
title: "Breaking Changes in PostSharp 6.4"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses a change in PostSharp 6.4 where the LocationInterceptionAspect.OnSetValue advice will now be invoked for initializers of static fields and properties."
---
# Breaking Changes in PostSharp 6.4


## LocationInterceptionAspect and initializers of static fields and properties

The <xref:PostSharp.Aspects.LocationInterceptionAspect.OnSetValue(PostSharp.Aspects.LocationInterceptionArgs)> advice will now be invoked for initializers of static fields and properties. These assignments were previously ignored. The breaking change does not affect binaries that have already been compiled. It only affects new compilations. 


