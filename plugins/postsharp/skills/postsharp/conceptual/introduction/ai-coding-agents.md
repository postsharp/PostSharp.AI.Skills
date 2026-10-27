---
uid: postsharp-and-ai
title: "Using PostSharp with AI Coding Agents"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming;AI"
summary: "Why aspects remain valuable when AI coding agents write much of the code, the problems an agent meets in a project that uses PostSharp, and the tools that PostSharp provides to solve them: a plugin with the documentation, the postsharp introspect commands, documented build messages, the aspect test framework, and architecture constraints."
---
# Using PostSharp with AI Coding Agents

AI coding agents such as Claude Code and OpenAI Codex now write a large part of the code of many projects. This article explains why aspects remain valuable in that situation, what an agent needs to know to work well in a project that uses PostSharp, and what PostSharp provides for it.

## Why aspects still matter when an agent writes the code

An agent can generate boilerplate quickly: the implementation of `INotifyPropertyChanged`, a logging statement at the start of every method, the caching of a result. It might seem that a tool that removes boilerplate is no longer needed. It still is, for three reasons:

* **Generated code must still be read.** Every line that an agent writes is a line that a person reviews and maintains, and that the next agent reads before it changes anything. One aspect replaces hundreds of repetitions of the same pattern, so the code to review, and the context that an agent must read, stays small.
* **Repeated code drifts.** When the same pattern is written a hundred times, by people or by agents, the copies start to differ: a property that does not raise its notification, a method that is not logged. An aspect applies the same behavior everywhere, and a change to the pattern is made in one place.
* **The compiler enforces the rules, not the prompt.** An instruction in a prompt is followed most of the time. An aspect, or an architecture constraint, is applied or verified by the compiler at every build, whoever wrote the code.

## What an agent gets wrong in a PostSharp project

An agent that knows C# but not PostSharp makes predictable mistakes:

* **It guesses the API.** PostSharp has two decades of API history, and many names that look plausible are obsolete or have never existed.
* **It reasons from the source code.** PostSharp changes the compiled assembly, not the source code. The source of a method does not show that a multicast attribute on the assembly applies a logging aspect to it, or that a threading model makes it thread-safe. An agent that reads only the source misses this behavior, or adds it a second time.
* **It hides problems to get a clean build.** To make a build pass, an agent may remove an aspect, disable a warning, or set `SkipPostSharp`, instead of fixing the cause of the message.

## What PostSharp provides for agents

### A plugin with the documentation

The PostSharp plugin for Claude Code and OpenAI Codex gives the agent this documentation and the complete API reference, with scripts to find an article or an API. It also gives the agent a few habits: look up the API before writing code, build to verify, read the build messages, and ask how aspects were applied instead of guessing. See <xref:ai-agents>.

### Inspection of the woven code

The `postsharp introspect` commands tell, from the files of the last build, which aspects apply to a declaration, what created them, and what they changed, in order. Their JSON output and exit codes are designed for agents:

```powershell
postsharp introspect show declaration src/Customer.cs:18
```

With this command, an agent can check that an aspect applies where it should before it explains a member, and after it changes an aspect or how it is applied. See <xref:introspect>.

### Documented build messages

PostSharp reports invalid aspect usages and failed constraints as build errors and warnings with codes, such as `PS0215` or `LA0231`. Each code has an article that explains its cause and how to fix it. An agent that reads the message and then the article fixes the cause instead of hiding the message. See <xref:messages> and <xref:errors>.

### Tests that an agent can run

The aspect test framework compiles, weaves and runs each test in an assembly of its own, and compares the result and the build messages with expected files. An agent that writes or changes an aspect can verify it with a test, without a person checking the behavior. See <xref:aspect-tests>.

### Guardrails for generated code

Architecture constraints verify the code at build time: which namespaces can use a type, which members can be called from where, which classes must follow a convention. They apply to the code that an agent writes as well as to the code that a person writes, and the agent sees a violation as a build error. See <xref:constraints>.

## A typical workflow

With the plugin and the `postsharp` tool installed, a task in a project that uses PostSharp goes as follows:

1. Before it changes a member, the agent runs `postsharp introspect show declaration` to learn which aspects apply to it.
2. It looks up the API and the article of the feature that it uses, rather than guessing.
3. It writes the code, then builds the project.
4. It reads the PostSharp messages of the build, and the article of each code, and fixes their causes.
5. It runs `postsharp introspect` again to check that the aspects apply where they should, and runs the tests.

## Recommendations for teams

* Install the PostSharp plugin and the `postsharp` tool on every machine where an agent works, including in containers and build agents. See <xref:ai-agents>.
* Keep the aspects of the team in a library, with tests that use the aspect test framework, so that an agent changes one aspect rather than many copies of a pattern.
* Turn the conventions that you would otherwise write in a prompt into architecture constraints, so that the compiler enforces them.
* Tell the agent not to disable PostSharp messages, `SkipPostSharp` or aspects without asking. The plugin already instructs it to fix the cause of a message.

## See Also

**Other Resources**

<xref:ai-agents>
<br><xref:introspect>
<br><xref:aspect-tests>
<br><xref:constraints>
<br><xref:messages>
<br>
