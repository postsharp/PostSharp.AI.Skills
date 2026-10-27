---
uid: ai-agents
title: "Configuring AI Coding Agents"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Install the PostSharp plugin for AI coding agents such as Claude Code and OpenAI Codex, so that they use the PostSharp documentation and API reference when they write code."
---
# Configuring AI Coding Agents

AI coding agents such as [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) and [OpenAI Codex](https://developers.openai.com/codex) write better PostSharp code when they can look up the real documentation instead of guessing from memory. The PostSharp plugin gives them the complete PostSharp documentation and API reference.

## What the plugin provides

The PostSharp plugin gives your AI agent access to:

- **Conceptual documentation**. All the articles of this documentation, including custom aspects, multicasting, architecture constraints, configuration, deployment, and licensing.
- **API reference**. The documentation of all PostSharp namespaces and types.
- **Pattern libraries**. The documentation of PostSharp Caching, Logging, Threading, Contracts, INotifyPropertyChanged, XAML, and undo/redo.
- **Instructions**. Guidance on looking up the API before writing code, on checking the build messages afterward, and on investigating how aspects were applied with `postsharp introspect`.

The plugin matches the version of the documentation it was built from. If your projects reference an older version of PostSharp, the agent is instructed to prefer the XML documentation of the referenced packages.

## Installing in Claude Code

1. Add the PostSharp marketplace to Claude Code:

   ```text
   /plugin marketplace add https://github.com/postsharp/PostSharp.AI.Skills
   ```

2. Install the PostSharp plugin:

   ```text
   /plugin install postsharp
   ```

To update to a newer version:

```text
/plugin update postsharp
```

## Installing in OpenAI Codex

1. Add the PostSharp marketplace to Codex:

   ```powershell
   codex plugin marketplace add postsharp/PostSharp.AI.Skills
   ```

2. Install the `postsharp` plugin from the plugin directory (**Plugins**) in the Codex app.

To update to a newer version:

```powershell
codex plugin marketplace upgrade
```

## Letting the agent investigate the woven code

PostSharp changes the compiled assembly, not the source code, so an agent that reads only the source does not see which aspects enhance a member. The plugin instructs the agent to ask the `postsharp introspect` commands, which read the files of the last build. Install the `postsharp` tool on the machine where the agent runs:

```powershell
dotnet tool install -g PostSharp.Tool
```

See <xref:introspect>.

## Using the plugin

Once the plugin is installed, the agent uses it when you ask about:

- Creating or modifying aspects
- Applying aspects with attribute multicasting
- Using the PostSharp Patterns libraries
- Configuring PostSharp builds or fixing PostSharp build errors

If the agent does not seem to use PostSharp-specific knowledge, mention "PostSharp" explicitly in your prompt.

### Example prompts

- "Create an aspect that logs method entry and exit."
- "Add caching to this service method, and invalidate it when the record is updated."
- "Make this view model implement INotifyPropertyChanged with PostSharp."
- "How do I apply an aspect to all public methods in a namespace?"
- "Which aspects apply to `Customer.FullName`, and in which order?"

## See Also

**Other Resources**

<xref:postsharp-and-ai>
<br><xref:introspect?text=Inspecting How Aspects Were Applied (postsharp introspect, for agents)>
<br>[Claude Code documentation](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)
<br>[OpenAI Codex plugin documentation](https://developers.openai.com/codex/plugins/build)
