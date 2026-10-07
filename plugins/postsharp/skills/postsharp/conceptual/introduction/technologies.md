---
uid: technologies
redirect_from:
  - introduction/overview/technologies
title: "Key Technologies"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp is a metaprogramming tool for .NET that uses Aspect-Oriented Programming (AOP) for program transformations. It also uses static and dynamic program analysis for structural analysis and run-time checks respectively."
---
# Key Technologies


## Metaprogramming

*Metaprogramming* is the writing of a program that analyzes and transforms itself or other programs. PostSharp internally represents a .NET program as a mutable .NET object model, so PostSharp can be considered a metaprogramming tool for .NET. 

However, general metaprogramming (the ability to perform arbitrary modifications on a program) is a highly complex discipline. Although it may seem easy to perform simple modifications on simple programs, it is actually much more difficult to implement non-trivial transformations that work in all cases. Metaprogramming can result in a decrease in productivity when used improperly and it is very difficult, for application developers who lack specific training in compilers and metaprogramming, to use it properly.

Since general metaprogramming is too complex and too low level, we need a higher layer of abstraction that makes it easier and safer to express program transformations. Essential qualities of this abstraction layer would include safe composition of several transformations on the same declaration and restrictions on changing the program semantics.

*Aspect-oriented programming* fulfills these qualities as a disciplined approach to metaprogramming. 


## Aspect-oriented programming

PostSharp Aspect Framework is built on the principle of *Aspect-Oriented Programming* (AOP), a well-established programming paradigm, orthogonal to (and non-competing with) object-oriented programming or functional programming, that allows developers to modularize the implementation of some features that would otherwise crosscut a large number of classes and methods. 

We can confidently say that PostSharp is the most advanced AOP framework for Microsoft .NET.

For details on PostSharp's implementation of AOP, see <xref:custom-aspects>. 


## Static program analysis

*Static program analysis* is the analysis of a program without executing it. 

There are two families of static analysis tools:

* *Structural static analysis* tools analyze the program's declarations and instructions, but do not attempt to understand the run-time behavior of the program. Microsoft Code Analysis belongs to this category. 

* *Behavioral static analysis* tools are based on iterative techniques like abstract interpretation, model checking or data-flow analysis. Behavioral static analysis is much more complex and time-consuming. Microsoft Code Contracts belong to this category. 

PostSharp contains tools for *structural* static analysis only. These tools consist of complete access to the `System.Reflection` model of the assembly being built, navigating through code relationships using the <xref:PostSharp.Reflection.ReflectionSearch> facility, and an expression tree decompiler. 

The most important use case for static analysis in PostSharp is when writing aspects, to determine how the target program should be transformed. Any of the static analysis tools can be used to build aspects. This contrasts with other AOP implementations like AspectJ, which defines its own specific language (*pointcut* language) to select target declarations. 

Aspects like <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> or threading models make advanced use of static analysis. 

A secondary role for static analysis is architecture validation. This role is marketed as the *PostSharp Architecture Framework*, which defines a notion of architectural constraint. See <xref:constraints> for more information. 


## Dynamic program analysis

*Dynamic program analysis* is the analysis of a program during its execution. Dynamic analysis is often used to detect issues early, before they cause bigger damage. A typical example of dynamic analysis is the one that occurs when an object is cast to a type: when the safety of the type conversion cannot be proved using static analysis, the type conversion must be verified at run time, and an <xref:System.InvalidCastException> is thrown when an invalid cast is detected. 



PostSharp uses dynamic analysis to check the program against threading models. Since many model properties cannot be reliably verified at build time, they must be enforced at run time. For instance, with the Synchronized threading model, accessing a field without owning access to the object would result in a <xref:PostSharp.Patterns.Threading.ThreadAccessException>. For details, see <xref:threading-models>. 

In PostSharp, dynamic analysis is achieved by adding instrumentation aspects to the program.


