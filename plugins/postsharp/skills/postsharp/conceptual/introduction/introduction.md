---
uid: introduction
title: "Introduction"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "This document introduces PostSharp, a tool designed to eliminate boilerplate code in object-oriented programming, enabling developers to work more efficiently and avoid bugs. The tool adds support for patterns into C# and VB languages."
---
# Introduction

In conventional object-oriented programming, technical concerns like threading, INotifyPropertyChanged or logging generally result in a large amount of boilerplate code. This boilerplate code is tangled with business code, and makes it more difficult to understand and modify the business meaning of the source code.

Boilerplate code is in fact instances of source code patterns. Like any pattern, its instances exhibit a great amount of regularity and predictability. Any decent software developer would be able to explain to a colleague how to properly implement INotifyPropertyChanged. These implementation guidelines would form some sort of algorithm, expressed in natural language, that the colleague would execute. Most of the work would be repetitive and would only require a limited amount of creativity.

Algorithmic work is exactly what machines are good at, so why not offload it to the compiler? This is exactly what PostSharp has been designed for. Traditional languages have concepts like classes, methods, fields, but they do not have any construct to represent patterns like the implementation of INotifyPropertyChanged.

PostSharp adds support for patterns into the C# and VB languages, allowing developers to work more productively at a higher level of abstraction and to avoid the bugs that stem from working with a large amount of technical details.


## In this chapter

| Chapter | Description |
|---------|-------------|
| <xref:what-is-postsharp> | This topic gives some small examples to help you understand what PostSharp is about. |
| <xref:benefits> | This chapter explains the main benefits of using PostSharp. |
| <xref:how-it-works> | This chapter proves that there is no magic involved, and that everything relies on stable specifications and documented extension points. |
| <xref:getting-started> | This chapter suggests a few learning resources according to your role in and the phase of the project. |
| <xref:whats-new> | This chapter lists the new features in each major and minor release. |
| <xref:postsharp-and-ai> | This topic explains why aspects remain valuable when AI coding agents write the code, and what PostSharp provides for these agents. |


