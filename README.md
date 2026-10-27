# PostSharp Plugin for AI Coding Agents

This repository contains the PostSharp plugin for [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) and [OpenAI Codex](https://developers.openai.com/codex/plugins/build). It gives coding agents the PostSharp documentation, so they write PostSharp code based on the real API instead of guessing from memory.

## Installation

### Claude Code

1. Add this marketplace to Claude Code:

   ```
   /plugin marketplace add https://github.com/postsharp/PostSharp.AI.Skills
   ```

2. Install the PostSharp plugin:

   ```
   /plugin install postsharp
   ```

### OpenAI Codex

1. Add this marketplace to Codex:

   ```
   codex plugin marketplace add postsharp/PostSharp.AI.Skills
   ```

2. Install the PostSharp plugin from the Codex plugin directory (**Plugins** in the Codex app).

## What's included

The plugin provides the coding agent with access to:

- **Conceptual documentation**: Guides on custom aspects, multicasting, architecture constraints, configuration, deployment, and licensing.
- **API reference**: Full documentation for all PostSharp namespaces and types.
- **Errors and warnings**: An article for each message that PostSharp reports at build time, with its cause and resolution.
- **Pattern libraries**: Documentation for PostSharp Caching, Logging, Threading, Contracts, INotifyPropertyChanged, XAML, and undo/redo.
- **Investigation of the woven code**: Instructions to ask the `postsharp introspect` commands which aspects apply to a member and what they changed, instead of guessing from the source.

## Usage

Once installed, the agent automatically uses PostSharp knowledge when you ask questions about:

- Creating or modifying aspects
- Applying aspects with attribute multicasting
- Using the PostSharp Patterns libraries
- Configuring PostSharp builds and fixing PostSharp build errors

### Example prompts

- "Create an aspect that logs method entry and exit"
- "Add caching to this service method, and invalidate it when the record is updated"
- "Make this view model implement INotifyPropertyChanged with PostSharp"
- "How do I apply an aspect to all public methods in a namespace?"

## Resources

- [PostSharp Documentation](https://doc.postsharp.net)
- [PostSharp Samples](https://github.com/postsharp/PostSharp.Samples)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)
- [OpenAI Codex Plugin Documentation](https://developers.openai.com/codex/plugins/build)

---

> **Note:** This repository is entirely generated from the PostSharp documentation. Do not edit files here directly.
