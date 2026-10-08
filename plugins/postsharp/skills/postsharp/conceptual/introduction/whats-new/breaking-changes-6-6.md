---
uid: breaking-changes-66
redirect_from:
  - introduction/whatsnew/breaking-changes-66
title: "Breaking Changes in PostSharp 6.6"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 6.6 has introduced changes such as counting lines of code instead of classes for premium features and replacing automatic switch to Developer Mode in PostSharp Logging with an opt-in approach. "
---
# Breaking Changes in PostSharp 6.6


## Number of Lines of Code vs Number of Types

PostSharp Essentials allowed to use premium features for up to 10 classes per project and 50 classes per solution, but we stopped counting classes and instead moved to count number of lines of code in classes. We have set this limit to 1,000 lines of code per solution.

Some users may be negatively affected by this change. If this is your case and you don't want to upgrade to a commercial edition, we recommend that you stay with the version 6.5, which is a Long-Term Support one.

For details about how we count lines of code, see <xref:licensing-counting-lines>. 


## PostSharp Logging - Developer Mode

In previous versions, PostSharp Logging would automatically switch to Developer Mode according to the installed license. We found this approach unreliable and replaced it by an explicit opt-in approach.

To enable development mode, set the `LoggingDeveloperMode` property to `true` in *postsharp.config*. See <xref:configuration-system> for details. 


