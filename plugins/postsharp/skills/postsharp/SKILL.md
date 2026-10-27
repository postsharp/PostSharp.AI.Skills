---
name: postsharp
description: Complete PostSharp documentation for aspect-oriented programming in C#, covering custom aspects (OnMethodBoundaryAspect, MethodInterceptionAspect, LocationInterceptionAspect, EventInterceptionAspect), attribute multicasting, architecture constraints, and the PostSharp Patterns libraries (caching, logging, threading models, contracts, INotifyPropertyChanged, XAML). Use whenever writing, reviewing, or debugging code in a project that references PostSharp packages; when a build reports PostSharp errors or warnings (PS, LA, SR, AR, COM, DOM, THR, XAM, CAC or DIA codes, each documented in an article); when configuring PostSharp MSBuild properties, postsharp.config, licensing, or deployment on a build server; or when maintaining an existing PostSharp code base, including explaining which aspects apply to a member and why woven code behaves differently from its source.
---

# PostSharp Documentation Skill

This skill contains the complete PostSharp documentation: conceptual articles and the API reference.

This skill pertains to PostSharp 2027.0.

> [!NOTE]
> PostSharp is maintained, but [Metalama](https://www.metalama.net) is its successor and is recommended for new projects. Use this skill for code bases that already use PostSharp. Do not suggest migrating to Metalama unless the user asks.

## Paths in this skill

All the paths in this file are relative to the base directory of this skill, which is the directory of this `SKILL.md` file. They are not relative to the working directory. Run the scripts by their full path, and give full paths to `grep`:

```bash
python <skill-dir>/scripts/find-api.py OnMethodBoundaryAspect
grep -i "caching" <skill-dir>/index.yml
```

The articles reference images that are not bundled with the skill. Rely on the text around them.

## Before you write code (read this first)

Four habits prevent most mistakes when writing or explaining PostSharp code. Do them in this order.

### 1. Look up the real API — don't guess signatures

The complete, authoritative PostSharp API is in this skill under `api/`. PostSharp has two decades of API history, and many names that look plausible are obsolete or never existed. **Before writing an aspect or using a pattern library, look up the type or member with `find-api`** (see [Looking Things Up](#looking-things-up)).

If the user's project references a **different** PostSharp version than this skill, prefer the actual referenced version. Its XML docs ship in the NuGet cache (e.g. `~/.nuget/packages/postsharp.redist/<package-version>/lib/**/*.xml`).

### 2. Read the article before using a pattern library

The PostSharp Patterns libraries (`PostSharp.Patterns.Caching`, `PostSharp.Patterns.Diagnostics`, `PostSharp.Patterns.Threading`, `PostSharp.Patterns.Model`, `PostSharp.Patterns.Xaml`) need more than an attribute: most require a package, a backend, or a configuration step at application startup. Find the getting-started article with `find-doc` and follow it rather than guessing. For example, caching requires `CachingServices.DefaultBackend` to be set, and logging requires `LoggingServices.DefaultBackend`.

### 3. Build to verify — PostSharp reports problems as build messages

PostSharp runs after the C# compiler and rewrites the compiled assembly. It reports invalid aspect usages, eligibility problems, and failed constraints as build errors and warnings with `PS` codes (for instance `PS0215`) or with codes defined by custom aspects. After writing or changing an aspect, run `dotnet build` (or `msbuild` for a non-SDK-style .NET Framework project) and read the diagnostics; treat a clean build as the pass condition.

**Every message that PostSharp or a pattern library reports has an article** at `conceptual/errors/<id>.md`, where `<id>` is the code in lowercase: `PS0215` is `conceptual/errors/ps0215.md`, and `python <skill-dir>/scripts/find-doc.py ps0215` finds it too. The article gives the message text, the cause, and the resolution. Read it before you change code to fix the message. `conceptual/errors/errors.md` explains the prefixes and lists the categories. The examples of the articles, an `<ID>_Error` program and an `<ID>_Fixed` program, are included from `code/samples/`, which this skill does not bundle; when you need one, read the live page `https://doc.postsharp.net/<id>`. A code without an article is either reported by a custom aspect, whose author defines it, or obsolete.

A clean build proves the aspects were applied; it does not prove behavior. When feasible, also **run the program** (or a test) to check that the enhanced code does what was asked. Because PostSharp modifies IL, the C# source never shows the aspect code. To see which aspects were applied, and where, ask `postsharp introspect` (habit 4); to see the woven IL itself, decompile the output assembly (for example with ILSpy or `ilspycmd`).

Observe these rules in the builds you run:

- **Never set `PostSharpAttachDebugger`.** It makes PostSharp wait for a debugger to attach, so the build hangs. Debugging the build is for a human (see `conceptual/framework/testing/debugging-build-time.md`).
- **Report licensing errors; do not work around them.** When the build reports a missing or invalid license (for instance `PS0242` or `PS0301`), tell the user and point to `conceptual/licensing/troubleshooting.md`. Do not invent a license key, and do not set `PostSharpLicense`. On Linux and macOS, an interactive build needs a registered license since PostSharp 2027.0.
- **Do not hide problems to get a clean build.** Do not set `SkipPostSharp`, add codes to `PostSharpDisabledMessages`, or remove aspects to make the build pass, unless the user asks. Fix the cause. When `TreatWarningsAsErrors` is set, PostSharp warnings fail the build too.

See `conceptual/framework/testing/` for testing aspects.

### 4. Ask what PostSharp did — don't infer it from the source

The source of a member does not show the aspects that apply to it: multicast attributes on the assembly or on a type, inherited attributes, aspect providers and XML configuration apply aspects from elsewhere. **Before you explain or change a member of a project that uses PostSharp, and after you change an aspect or how it is applied, query the last build with `postsharp introspect`:**

```bash
postsharp introspect show declaration src/Customer.cs:18
```

It lists the aspect instances of the declaration, its accessors and backing field, what created each one (predecessor kind `Attribute`, `Multicast`, `Inherited`, `Xml`, or `ChildAspect` for an aspect provider, whose aspect instance is the predecessor's `aspectInstance`), and the transformations in order. The line is the line of the *name* of the declaration.

- **Order:** for the transformations of one declaration, a lower `order` is **outer**: it runs first on entry and last on exit. Orders compare only within one project and target framework.
- **Install it when `postsharp` is not found:** `dotnet tool install -g PostSharp.Tool`. It reads the files of projects built with PostSharp 2023.0 and later, so the project does not have to be upgraded.
- **It never builds.** It reads the files of the last build. Exit code 2 means a file is missing: build the project, then query again. A warning on the standard error says when a file predates a change to the source; `list files` shows `stale` for each file.
- **Several questions: one call.** `batch` runs one command per line over one load of the files:

  ```bash
  postsharp introspect batch <<'EOF'
  show declaration src/Customer.cs:18
  list aspect-instances --where "aspectClass/shortName eq 'Log'" --count
  EOF
  ```

- **Output is limited.** `list` writes at most 100 objects and `show` at most 20 per list of related objects. Each list says `count` (the total) and `truncated`; a truncated one adds `byNamespace` and, in `show`, `next`, the exact command that lists all its objects. Follow `next`, narrow with `--where`, or add `--all` only when you need everything. The related objects of `show` (but `show declaration`) are summary rows: `show` one of them for details.
- **Drill down, one level at a time:** start from `list aspect-classes` or `show declaration <file>:<line>`; take the `id` of a summary row (any unique prefix works) and `show aspect-instance <id>` for its predecessors and transformations, then `show predecessor <id>` for everything that one attribute or multicast created. For a truncated list, read `count` and `byNamespace`, then run `next`, narrowed with an extra `--where` condition when it is large. Identifiers are stable across builds while the source is unchanged.
- **Other queries:** `list <collection> --where "<OData $filter condition>"` (the full grammar: functions such as `contains` or `matchesPattern`, `any`/`all`, `in`, `$count`, arithmetic; put `$count` in single quotes in bash and PowerShell) over `aspect-classes`, `aspect-instances`, `predecessors`, `transformations`, `annotations`, `declaration-mappings` and `declarations`; `show aspect-instance|predecessor|aspect-class <selector>`; `metrics` for the lines of code saved (`savedRatio` = saved / (hand-written + saved), `avoidedRatio` = saved / hand-written; `postsharp introspect metrics --help` defines every field). `postsharp introspect schema` lists the fields you can use in `--where` and `--select`.
- **The scope** is the only project or solution of the current directory, or `--project` / `--solution`. In a solution, the files of all projects and target frameworks are merged. A project that compiles another project's source files has its own aspect instances on the same declarations: count distinct `targetDeclaration/id` to count source declarations.
- Exit code 3 is an ambiguous selector, and 4 an invalid query or data that PostSharp does not record (diagnostics, skipped aspect instances).

See `conceptual/framework/testing/introspect.md`.

## Directory Structure

| Directory | Contents |
|-----------|----------|
| `conceptual/introduction/` | Overview, benefits, getting started, what's new and breaking changes per version |
| `conceptual/framework/` | Writing custom aspects (`aspects/`), architecture constraints (`architecture/`), testing and debugging (`testing/`) |
| `conceptual/applying-aspects/` | Applying aspects: attributes, multicasting, inheritance, `IAspectProvider`, XML |
| `conceptual/caching/` | Caching library, its backends (memory, Redis), and the synchronization of local caches across servers (Redis, Azure Service Bus) |
| `conceptual/logging/` | Logging library and its backends (Serilog, NLog, log4net, ETW, Application Insights...) |
| `conceptual/threading/` | Threading models (immutable, freezable, synchronized, reader-writer, actor, thread-affine), dispatching, deadlock detection |
| `conceptual/contracts/` | Validating parameters, fields and properties |
| `conceptual/inotifypropertychanged/` | `[NotifyPropertyChanged]` and UI framework integration |
| `conceptual/aggregatable/` | Parent-child relationships, `[Aggregatable]`, `[Disposable]` |
| `conceptual/xaml/` | Dependency properties, attached properties, commands |
| `conceptual/deployment/`, `conceptual/licensing/`, `conceptual/configuration/` | Installation, requirements, build servers, licensing, MSBuild properties, `postsharp.config`, build messages |
| `conceptual/errors/` | One article per error or warning code (`<id>.md`, lowercase), the index `errors.md`, and one page per category (`errors-<category>.md`) |
| `conceptual/deprecated/` | Deprecated features, such as undo/redo. Do not use them in new code |
| `api/` | API reference YML files from DocFx (trimmed to the documentation content), indexed by `api/.manifest` |
| `scripts/` | Helper scripts: `find-api` (API lookup) and `find-doc` (article search), each in Python and PowerShell |
| `index.yml` | Table of contents of the articles, in reading order, with their uid, path and summary |

## Looking Things Up

Use the two helper scripts. Each has a Python version and a PowerShell version, which works on Windows PowerShell 5.1 and pwsh. Prefer Python when it is available.

| To find | Run |
|---------|-----|
| Articles about a topic | `python <skill-dir>/scripts/find-doc.py caching invalidation` |
| The article of an `<xref:uid>` link | `python <skill-dir>/scripts/find-doc.py <uid>` |
| A type or member, by name or uid | `python <skill-dir>/scripts/find-api.py OnMethodBoundaryAspect` |
| The same, without Python | `powershell -File <skill-dir>/scripts/find-doc.ps1 ...` and `powershell -File <skill-dir>/scripts/find-api.ps1 ...` |

`find-doc` requires every keyword to appear in the name, uid, path or summary of an article. A single keyword that equals the uid of an article returns only that article. It prints the path of each article to read.

`find-api` tries an exact uid first, then a case-insensitive substring. It prints the matching uids and the documentation of the most specific ones, so reading the whole YML file is rarely needed. In PowerShell, pass a generic uid, which contains a backtick, in single quotes.

Articles link to other articles and to APIs with `<xref:uid>`, or with `[text](xref:uid#anchor)`, where the anchor is a heading of the target. Resolve a uid that starts with `PostSharp.` with `find-api`; a trailing `*` (all the overloads of a member) is ignored. Resolve any other uid with `find-doc`.

Without Python or PowerShell, grep the indexes. Each entry of `index.yml` lists the name, then the uid, path and summary, so `-A3` shows the path:

```bash
grep -i -A3 "caching" <skill-dir>/index.yml
grep -i "OnMethodBoundaryAspect" <skill-dir>/api/.manifest
```

## Key Entry Points

| Topic | File | Description |
|-------|------|-------------|
| Getting Started | `conceptual/introduction/getting-started/getting-started.md` | Adding PostSharp to a project |
| Custom Aspects | `conceptual/framework/aspects/aspects.md` | Overview of aspect development |
| Simple Aspects | `conceptual/framework/aspects/simple-aspects/simple-aspects.md` | The aspect classes and what they do |
| Method Decorator | `conceptual/framework/aspects/simple-aspects/method-decorator.md` | `OnMethodBoundaryAspect`: code before and after a method |
| Method Interception | `conceptual/framework/aspects/simple-aspects/method-interception.md` | `MethodInterceptionAspect`: wrap or replace a method |
| Property and Field Interception | `conceptual/framework/aspects/simple-aspects/property-interception.md` | `LocationInterceptionAspect` |
| Applying Aspects | `conceptual/applying-aspects/applying-aspects.md` | Attributes, multicasting, inheritance, `IAspectProvider` |
| Aspect Lifetime | `conceptual/framework/aspects/lifetime.md` | Build-time instantiation, serialization, static and per-instance scopes |
| Aspect Dependencies | `conceptual/framework/aspects/aspect-dependencies/aspect-dependencies.md` | Ordering several aspects on the same declaration |
| Testing and Debugging | `conceptual/framework/testing/testing.md` | Testing run-time and build-time logic |
| Inspecting Applied Aspects | `conceptual/framework/testing/introspect.md` | `postsharp introspect`: which aspects apply where, what created them, what they changed |
| Architecture Constraints | `conceptual/framework/architecture/architecture.md` | Architecture validation |
| Caching | `conceptual/caching/getting-started.md` | Method result caching |
| Logging | `conceptual/logging/getting-started.md` | Adding logging to a project |
| Threading Models | `conceptual/threading/models/models.md` | Choosing and applying a threading model |
| Contracts | `conceptual/contracts/contracts.md` | Validating parameters, fields and properties |
| INotifyPropertyChanged | `conceptual/inotifypropertychanged/inotifypropertychanged.md` | `[NotifyPropertyChanged]` and its dependency analysis |
| MSBuild Properties | `conceptual/configuration/msbuild-properties.md` | All PostSharp MSBuild properties |
| Build Servers | `conceptual/deployment/build-server.md` | Using PostSharp on a build server |
| Errors and Warnings | `conceptual/errors/errors.md` | The prefixes of the message codes, and the categories of the articles |

## Common Patterns

### Aspect Classes

| Base Class | Target | Key Overrides | Use When |
|------------|--------|---------------|----------|
| `OnMethodBoundaryAspect` | Methods | `OnEntry`, `OnSuccess`, `OnException`, `OnExit` | Run code before and after a method, without controlling its invocation |
| `OnExceptionAspect` | Methods | `OnException`, `GetExceptionType` | Handle exceptions |
| `MethodInterceptionAspect` | Methods | `OnInvoke` (call `args.Proceed()`) | Wrap, retry, or replace the invocation |
| `LocationInterceptionAspect` | Fields, properties | `OnGetValue`, `OnSetValue` | Intercept property or field access |
| `EventInterceptionAspect` | Events | `OnAddHandler`, `OnRemoveHandler`, `OnInvokeHandler` | Intercept event subscription and raising |
| `InstanceLevelAspect`, `TypeLevelAspect` | Types | Advice attributes (`[IntroduceMember]`, `[OnMethodEntryAdvice]`...) | Introduce members or interfaces, composite aspects |
| `AssemblyLevelAspect` | Assemblies | `IAspectProvider.ProvideAspects` | Apply aspects programmatically |

These live in `PostSharp.Aspects`. Look up each base class with `find-api` before overriding it.

```csharp
[PSerializable]
public sealed class LogAttribute : OnMethodBoundaryAspect
{
    public override void OnEntry( MethodExecutionArgs args )
    {
        Console.WriteLine( $"Entering {args.Method.Name}" );
    }

    public override void OnExit( MethodExecutionArgs args )
    {
        Console.WriteLine( $"Leaving {args.Method.Name}" );
    }
}
```

### Applying Aspects

- On one declaration: add the attribute, e.g. `[Log]`.
- On many declarations: multicast it, e.g. `[assembly: Log( AttributeTargetTypes = "MyApp.Services.*" )]`, or put it on a type to apply it to all its members. Filter with `AttributeTargetMemberAttributes`, `AttributeTargetElements`, and `AttributeExclude`. See `conceptual/applying-aspects/attributes/multicasting.md`.
- Programmatically: implement `IAspectProvider`.

### Common Pitfalls

| Mistake | Correct Approach |
|---------|------------------|
| Forgetting `[PSerializable]` on an aspect class | Aspects are instantiated at build time and serialized into the assembly. Mark aspect classes `[PSerializable]`, and mark fields that must not be serialized `[PNonSerialized]` |
| Initializing run-time state in the constructor | The constructor runs at build time. Use `CompileTimeInitialize` for build-time work and `RuntimeInitialize` for run-time work. See `conceptual/framework/aspects/lifetime.md` and `initialization.md` |
| Validating aspect usage in the constructor or at run time | Override `CompileTimeValidate` and report errors with `Message.Write`, so that misuses fail the build |
| Using `OnMethodBoundaryAspect` to skip or retry the method | Use `MethodInterceptionAspect`; `OnMethodBoundaryAspect` cannot control the invocation |
| Applying a pattern attribute without its runtime setup | Caching, logging and some threading models need backend or startup configuration. Read the getting-started article of the library |
| Adding the `PostSharp` package only to the project that defines the aspects | Every project where an aspect is *applied* must reference the `PostSharp` package (or a pattern package that depends on it), because PostSharp runs in that project's build |
| Guessing MSBuild property names | Read `conceptual/configuration/msbuild-properties.md` |

## API Reference

The `api/` directory contains DocFx-generated YML files for all public APIs. The files are trimmed to documentation content (`items:` with summary, syntax, parameters, returns); the DocFx `references:` rendering section is stripped at build time.

`api/.manifest` is a JSON index that maps every uid of a type or member to its YML file:

```json
"PostSharp.Aspects.OnMethodBoundaryAspect": "PostSharp.Aspects.OnMethodBoundaryAspect.yml"
"PostSharp.Aspects.OnMethodBoundaryAspect.OnEntry(PostSharp.Aspects.MethodExecutionArgs)": "PostSharp.Aspects.OnMethodBoundaryAspect.yml"
```

File naming conventions:

- Types: `Namespace.TypeName.yml`
- Generic types: backtick becomes dash, e.g., `AdvisableCollection<T>` → `PostSharp.Patterns.Collections.AdvisableCollection-1.yml`
- All members of a type are in the same YML file as the type

### YML File Structure

Each type's YML file contains **all members** of that type in a single file:

```yaml
items:
- uid: Namespace.TypeName           # Type definition
  commentId: T:Namespace.TypeName
  type: Class|Interface|Enum|...
  summary: Type description
  remarks: Detailed explanation
  syntax:
    content: public class TypeName : BaseClass
  children:                         # List of member UIDs
  - Namespace.TypeName.Method1

- uid: Namespace.TypeName.Method1   # Member definition
  commentId: M:Namespace.TypeName.Method1
  type: Method|Property|Field|...
  summary: Member description
  syntax:
    content: public void Method1()
  parameters: [...]                 # For methods
  return: { type: ..., description: ... }
```

### Key Namespaces

| Namespace | Purpose |
|-----------|---------|
| `PostSharp.Aspects` | Aspect base classes and their `*Args` classes |
| `PostSharp.Aspects.Advices` | Advice attributes for composite aspects |
| `PostSharp.Aspects.Dependencies` | Ordering and dependencies between aspects |
| `PostSharp.Extensibility` | Multicasting, `MulticastAttribute`, build messages |
| `PostSharp.Serialization` | `[PSerializable]` and the PostSharp serializer |
| `PostSharp.Constraints` | Architecture constraints |
| `PostSharp.Patterns.Caching` | Caching aspects, backends, invalidation |
| `PostSharp.Patterns.Diagnostics` | Logging aspects and backends |
| `PostSharp.Patterns.Threading` | Threading models, dispatching, deadlock detection |
| `PostSharp.Patterns.Model` | `[NotifyPropertyChanged]`, aggregatables, weak events |
| `PostSharp.Patterns.Contracts` | Contract attributes (`[NotNull]`, `[Required]`, `[Range]`...) |
| `PostSharp.Patterns.Recording` | Undo/redo |
| `PostSharp.Patterns.Xaml` | Dependency properties, attached properties, commands |

## Linking to Documentation

When referencing documentation articles, provide links to the live documentation at `https://doc.postsharp.net`.

The simplest link to an article is its uid: `https://doc.postsharp.net/<uid>`, for instance `https://doc.postsharp.net/caching-invalidation`. The site redirects every article uid to the page of the article.

The canonical URL is `https://doc.postsharp.net/<path>`, where `<path>` is derived from the file path under `conceptual/`:

- Remove the `conceptual/` prefix and the `.md` suffix.
- Use lowercase, and replace `.`, `_` and backticks with `-`.
- For a file whose name matches its parent folder (e.g., `path/leaf/leaf.md`), and for `index.md`, use the folder path: `path/leaf`.

| File Path | URL |
|-----------|-----|
| `conceptual/introduction/benefits/benefits.md` | `https://doc.postsharp.net/introduction/benefits` |
| `conceptual/caching/invalidation.md` | `https://doc.postsharp.net/caching/invalidation` |
| `conceptual/framework/aspects/simple-aspects/method-interception.md` | `https://doc.postsharp.net/framework/aspects/simple-aspects/method-interception` |

For an API, use `https://doc.postsharp.net/api/<type-uid>`, in lowercase, with dots and the backtick of a generic type replaced by dashes. For instance, `PostSharp.Aspects.OnMethodBoundaryAspect` is `https://doc.postsharp.net/api/postsharp-aspects-onmethodboundaryaspect`, and `AdvisableCollection<T>` is `https://doc.postsharp.net/api/postsharp-patterns-collections-advisablecollection-1`. A member is documented on the page of its type.

## External Resources

- Live documentation: https://doc.postsharp.net
- Samples: https://github.com/postsharp/PostSharp.Samples
- Support: https://www.postsharp.net/support
