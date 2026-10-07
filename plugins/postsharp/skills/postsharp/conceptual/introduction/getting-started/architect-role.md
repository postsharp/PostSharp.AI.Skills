---
uid: getting-started-architecture
redirect_from:
  - introduction/gettingstarted/getting-started-architecture
title: "Architecture Role: Selecting and Creating Aspects"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "This document is a guide to understanding the architecture role in adopting PostSharp, a tool that offers pre-built aspects and allows users to create custom aspects for specific needs."
---
# Architecture Role: Selecting and Creating Aspects

The first step in the process of adopting PostSharp is typically to understand *what* the product can do for you and *why* you should use (or not use) its features. This activity is typically part of the architecture role. 

As you will see, PostSharp offers a set of pre-built aspects implementing some of the most common patterns. As an architect, you will need to understand what these aspects can do for you and how they could fit and simplify your architecture.

However, standard patterns are only the beginning. To cover your specific needs, PostSharp includes construction kits that allow you to build your own pattern automation, namely the PostSharp Aspect Framework and the PostSharp Architecture Framework. Determining the need for custom aspects or architecture validation rules is typically also a part of the architecture role.

In a typical team, only a few people must be able to create custom aspects or architecture rules. These people must have a deeper understanding of PostSharp than the developers who will only use existing aspects and rules. This is why this skill set is included in the current section.

> [!NOTE]
> When writing this section, we realized that the current documentation has some serious weaknesses regarding conceptual and architectural materials. This is why we also refer to other resources hosted on our website.


## Introduction

Understanding the principles behind PostSharp will give you a foundation to build on. All patterns and techniques used by PostSharp relate back to this foundation.

| Topic | Articles |
|-------|----------|
| About PostSharp | <xref:benefits><br><xref:how-it-works><br><xref:requirements> |
| More About Design Pattern Automation | [Article: Design Pattern Automation](https://www.postsharp.net/downloads/documentation/Design%20Pattern%20Automation.pdf) |
| More About Aspect-Oriented Programming | [Aspect-Oriented Programming in Microsoft .NET](https://www.postsharp.net/aop.net)<br>[White Paper: Producing High-Quality Software with Aspect-Oriented Programming](https://www.postsharp.net/downloads/documentation/Producing%20High-Quality%20Software%20with%20Aspect-Oriented%20Programming.pdf) |


## Selecting pre-built pattern implementations

PostSharp offers a number of different pre-built patterns. The following documentation outlines how to use each of the available patterns.

| Topic | Articles |
|-------|----------|
| General patterns | <xref:contracts><br><xref:aggregatable> |
| User interface patterns | <xref:inotifypropertychanged-conceptual><br><xref:dependency-property><br><xref:command> |
| Multithreading | [White Paper: Threading Models for Object-Oriented Programming](https://www.postsharp.net/downloads/documentation/Threading%20Models%20for%20OOP.pdf)<br><xref:ui-dispatching><br><xref:background-dispatching> |
| Diagnostics | <xref:logging> |


## Creating automation for custom patterns

PostSharp's built-in patterns do not cover all scenarios in your codebase that can benefit from AOP. Learn how to build custom patterns using the same foundational components as are used for the built-in patterns.

| Topic | Articles |
|-------|----------|
| Aspects | <xref:custom-aspects> |
| Architecture Validation | <xref:constraints> |


