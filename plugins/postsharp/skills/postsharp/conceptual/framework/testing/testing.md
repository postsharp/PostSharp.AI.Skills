---
uid: testing-aspects
redirect_from:
  - custompatterns/aspects/testing/testing-aspects
title: "Testing and Debugging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses testing and debugging techniques for aspects in PostSharp, which differ from normal class libraries due to their unique characteristics."
---
# Testing and Debugging

Aspects should be tested like any piece of code. However, testing techniques for aspects differ from testing techniques for normal class libraries for several reasons:

* Aspect instantiation is not controlled by the user.

* Aspects partially execute at build time.

* Aspects can emit build errors. Logic that emits build errors should be tested too.

These characteristics are no obstacle to proper testing of aspects. The aspect test framework of the `PostSharp.Testing.AspectTesting` package weaves and runs each test in an assembly of its own, so it can test both the run-time behavior of an aspect and the errors and warnings that it emits at build time. Use it for most of your tests, and complement it with ordinary unit tests where they are more convenient.

## In this chapter

| Section | Description |
|---|---|
| <xref:aspect-tests> | Explains how to test aspects with the aspect test framework, which compiles, weaves, verifies and runs each test file in an assembly of its own. |
| <xref:aspect-test-reference> | Lists the test directives, MSBuild properties and other settings of the aspect test framework. |
| <xref:simple-tests> | Explains how to test the run-time behavior of an aspect with an ordinary unit test project. |
| <xref:testing-application> | Shows how to test that an aspect has been applied to the expected set of code artifacts. |
| <xref:consuming-dependencies> | Describes several ways for aspects to consume dependencies from dependency-injection containers and service locators. |
| <xref:debugging-runtime> | Explains how to debug run-time logic. |
| <xref:debugging-buildtime> | Explains how to debug build-time logic. |
| <xref:in-process-memory-leaks> | Explains why build-time code can keep a compilation in memory when several compilations share a process, and how to find and fix the leak. |
