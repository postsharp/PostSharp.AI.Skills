---
uid: how-it-works
redirect_from:
  - introduction/overview/how-it-works
title: "How PostSharp Works"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp is an extension to the C# or VB compiler that integrates into the build process, post-processing the compiler's output. It uses MSIL technology to transform and validate the intermediate assembly."
---
# How PostSharp Works
On a conceptual level, you can think of PostSharp as an extension to the C# or VB compiler. Practically, Microsoft's compilers themselves are not extensible, but the build process can be easily extended. This is what PostSharp does: it inserts itself into the build process and post-processes the output of the compiler.


## MSBuild integration

PostSharp integrates itself into the build process thanks to *PostSharp.targets*, which NuGet imports automatically into each project that references the *PostSharp* package. *PostSharp.targets* adds a few steps to the build process. The principal step is the post-processing of the compiler's output by PostSharp itself. 

See <xref:configuration-msbuild> for details. 


## MSIL rewriting

PostSharp post-processes the compiler output by reading and disassembling the intermediate assembly, executing the required transformations and validations, and writing the final assembly back to disk.

Although this might sound magical or dangerous, PostSharp's MSIL technology is stable and mature, and has been used by tens of thousands of projects since 2004. Other .NET products relying on MSIL transformation or analysis include Microsoft Code Contracts, Microsoft Code Analysis, and Microsoft Code Coverage.


