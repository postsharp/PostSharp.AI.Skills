---
uid: introspect
title: "Inspecting How Aspects Were Applied"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The postsharp introspect commands query the files of the last build to tell which aspects apply to a declaration, what created them, what they changed and in which order, how many lines of code they saved, for a project or a solution. They are designed for AI coding agents as much as for people."
---
# Inspecting How Aspects Were Applied

PostSharp changes the compiled assembly, not the source code. The source of a method does not show which aspects enhance it, what each aspect does to it, or why an aspect applies there at all, for instance because a multicast attribute on the assembly matches it. The `postsharp introspect` commands answer these questions from the files of the last build, for a project or a whole solution.

The commands are designed for AI coding agents as much as for people: their JSON output has a stable schema, their exit codes tell what went wrong, and they never build anything. Before an agent explains or changes a member of a project that uses PostSharp, it should run `postsharp introspect show declaration <file>:<line>`.

> [!NOTE]
> These commands are the command-line way to inspect how aspects were applied. In Visual Studio, Visual Studio Tools for Metalama and PostSharp show the same information in the editor: CodeLens and tooltips list the aspects of each declaration, and the Metrics window shows the lines of code saved. See <xref:installation>.

## What the commands read

When it builds a project, PostSharp writes two files next to the output assembly:

* the weaving symbol file, `<assembly>.pssym`, which records the aspect classes, the aspect instances, what created each aspect instance (its predecessors), the transformations in order, the annotations, the declaration mappings and the line counts;
* the symbol location database, `<assembly>.pspdb`, which records where each declaration is in the source code.

The commands locate these files by evaluating the projects with the MSBuild of a .NET SDK, through the `PostSharpSymbolFile` and `PostSharpSymbolLocationFile` properties. They also work on projects that are not SDK-style: such a project has the target framework of its `TargetFrameworkVersion`, for instance `net472` for `v4.7.2`.

### Compatibility with earlier versions of PostSharp

The tool reads the files of projects built with earlier versions of PostSharp, back to PostSharp 2023.0, so a project does not have to be upgraded to be queried. These versions write the same two files and set the same properties, and the commands give the same results for them: the same aspect classes, aspect instances, predecessors, transformations, source positions and metrics. This was checked with the last release of each version from 2023.0 to 2026.0.

The weaving symbol file does not record how a predecessor created its aspect instances, whatever the version of PostSharp. The commands infer it from the declarations, and set `isKindInferred` (see below). Earlier versions were not tested.

When no .NET SDK can be loaded, the commands search the build outputs instead: `bin/<configuration>/<target framework>` under the project, and `artifacts/bin/<project>/<configuration>_<target framework>` of the artifacts output layout, in the directory of the project or above it. The output of a runtime identifier or of a publish is left out when the build output exists. Without an evaluation, a file older than the project file is reported as stale, but a change of a source file is not detected.

The evaluations of the projects and an index of each file are cached in the local application data directory of the user (`%LOCALAPPDATA%\SharpCrafters\Introspection` on Windows), so that a second command does not evaluate the projects again. A cached evaluation is used until one of its inputs changes: the project file, a file that it imports, including the targets of a package that a restore brings back, a source file added to or removed from the project directory, the solution, `global.json`, or the .NET SDK. The index of a file is rebuilt when the file changes.

## Installing the tool

The commands are part of the `postsharp` command line tool, a .NET global tool:

```powershell
dotnet tool install -g PostSharp.Tool
```

The tool references no compiler assembly, and it reads files only. It requires the .NET 10 runtime or later.

The commands are now available as `postsharp introspect`. Build the project, then query it. To list the commands:

```powershell
postsharp introspect --help
```

`--help` after a command shows its arguments and options, for instance `postsharp introspect list --help`.

> [!WARNING]
> `postsharp introspect` does not build or rebuild the project. It reads the output of the **last build**, which may not describe the current source code. When the file of a project is missing, the command fails with exit code 2 and says which project to build. When a source file is newer than the file, the command still answers from the last build, but warns that the result is stale. Build the project after changing its code, then query it again.

## Scope: project or solution

As the `dotnet` command line does, the commands take the only project or solution file of the current directory. Use `--project <path>` or `--solution <path>` when the directory holds none or several.

* A project with several target frameworks has one file per target framework. The commands merge them and say, for each result, in which target frameworks it appears. `-f <tfm>` restricts the query to one target framework; a framework that the project does not target is an error that lists those it does.
* In a solution, the commands merge the files of every project, so an aspect class declared in a library and applied in an application is one aspect class.
* A project that compiles the source files of another project, for instance with `<Compile Include="..\Other\**\*.cs" />`, weaves its own aspect instances on the same source declarations. A solution then counts each of them once per project. To count source declarations rather than aspect instances, compare `targetDeclaration/id`, which is the same in both projects; `project` and `targetDeclaration/qualifiedId` tell them apart.
* `-c <configuration>` selects the build output of a configuration other than the default one, usually `Debug`.

## Showing a declaration

`show declaration` tells everything that applies to one declaration, its accessors and its backing field: the aspect instances with their predecessors, the transformations in order, the annotations and the declaration mappings.

```powershell
postsharp introspect show declaration Orders/Repository.cs:10
postsharp introspect show declaration App.Orders.Repository.Save
postsharp introspect show declaration "M:App.Orders.Repository.Save(App.Customer)"
```

With `--format text`, for a person at a terminal, the first command writes:

```text
Method App.Orders.Repository.Save(App.Customer) at Orders/Repository.cs:10:21 (App).

Aspect Log [f50384e0ebfe] on method App.Orders.Repository.Save(App.Customer).
  Created by the multicast attribute [0374f134812d] on App (kind inferred).
  Transformations, in order:
    2. App.Orders.Repository.Save(App.Customer): Wrapped by advice(s) OnEntry

From bin/Debug/net10.0/App.pssym (App, net10.0, Debug), written 2026-10-23 14:06:56 UTC.
```

By default, the output is JSON, which an AI agent reads: the same information with its identifiers:

```json
{
  "declaration": {
    "id": "M:App.Orders.Repository.Save(App.Customer)",
    "qualifiedId": "[App]M:App.Orders.Repository.Save(App.Customer)",
    "kind": "Method",
    "name": "Save",
    "namespace": "App.Orders",
    "assembly": "App",
    "project": "App",
    "targetFrameworks": [
      "net10.0"
    ],
    "location": "Orders/Repository.cs:10:21",
    "declaringType": {
      "id": "T:App.Orders.Repository",
      "kind": "Type"
    }
  },
  "relatedDeclarations": {
    "count": 0,
    "truncated": false,
    "items": []
  },
  "aspectInstances": {
    "count": 1,
    "truncated": false,
    "items": [
      {
        "id": "f50384e0ebfe",
        "aspectClass": {
          "name": "Lib.LogAttribute",
          "shortName": "Log"
        },
        "targetDeclaration": {
          "id": "M:App.Orders.Repository.Save(App.Customer)",
          "kind": "Method"
        },
        "project": "App",
        "targetFrameworks": [
          "net10.0"
        ],
        "predecessors": {
          "count": 1,
          "truncated": false,
          "items": [
            {
              "id": "0374f134812d",
              "kind": "Multicast",
              "isKindInferred": true,
              "declaration": {
                "id": "A:App",
                "kind": "Assembly"
              },
              "attributeClass": {
                "name": "Lib.LogAttribute",
                "shortName": "Log"
              },
              "project": "App",
              "targetFrameworks": [
                "net10.0"
              ]
            }
          ]
        },
        "transformations": {
          "count": 1,
          "truncated": false,
          "items": [
            {
              "aspectInstance": {
                "id": "f50384e0ebfe",
                "aspectClass": {
                  "name": "Lib.LogAttribute",
                  "shortName": "Log"
                },
                "targetDeclaration": {
                  "id": "M:App.Orders.Repository.Save(App.Customer)",
                  "kind": "Method"
                },
                "project": "App"
              },
              "targetDeclaration": {
                "id": "M:App.Orders.Repository.Save(App.Customer)",
                "kind": "Method"
              },
              "adviceDeclaration": {
                "id": "T:Lib.LogAttribute",
                "kind": "Type"
              },
              "description": "Wrapped by advice(s) OnEntry",
              "order": 2
            }
          ]
        },
        "tags": [],
        "linesOfCodeAvoided": 2
      }
    ]
  },
  "annotations": {
    "count": 0,
    "truncated": false,
    "items": []
  },
  "declarationMappings": {
    "count": 0,
    "truncated": false,
    "items": []
  }
}
```

* A position is `<file>:<line>`, where the line is the line of the **name** of the declaration: the symbol location database records where names are, not where bodies are, so a line inside a method body selects nothing. The path is relative to the current directory, or absolute, such as the path that an editor copies. When no aspect applies to the declaration on the line, the command shows the type that contains it. When no aspect applies to the type either, the command says so, with exit code 0: in JSON, `declaration` is `null` and `message` says why.
* A name is a C# name, such as `App.Customer.Name`, or a documentation comment identifier, such as `P:App.Customer.Name`. An ambiguous name lists the candidates (exit code 3).
* `show aspect-instance <id>`, `show predecessor <id>` and `show aspect-class <name>` show the other objects. An identifier is 12 hexadecimal characters, of which any unique prefix is accepted; the structural key of the object is accepted too. The key, such as `multicast:[App]A:App:Lib.LogAttribute`, describes the object in words, and its identifier is its hash; it is written only with `--select key`. An aspect class is named by its short name, such as `Log`, or by its full name when two classes have the same short name.
* `show` takes these four kinds of objects, which have an identifier or a name; `schema` marks them as `showable`. Transformations, annotations and declaration mappings are listed with `list` and `--where`.

### Reading the order and the predecessors

The `order` of a transformation is the order in which PostSharp wove it. For the transformations of one declaration, a lower order is **outer**: it runs first when the declaration is entered, and last when it exits. For instance, when a method has an interception aspect that dispatches it to the UI thread at order 558, and an exception handling aspect at order 559, the method is dispatched first, and the exception handler runs inside, on the UI thread. Orders are comparable within one project and target framework only.

The `kind` of a predecessor says what created its aspect instances:

| Kind | Created by |
|---|---|
| `Attribute` | A custom attribute on the target declaration. |
| `Multicast` | A multicast attribute on an enclosing declaration or on the assembly. See <xref:attribute-multicasting>. |
| `Inherited` | An attribute inherited from a base type or an overridden member. |
| `Xml` | An element of a configuration file. |
| `ChildAspect` | An aspect provider (<xref:PostSharp.Aspects.IAspectProvider>). The `aspectInstance` field of the predecessor is the aspect instance of the provider, which the source of the target declaration does not show. |

The weaving symbol file does not record the kind, so the commands infer it and set `isKindInferred`: an attribute on the target declaration is `Attribute`, one on a declaration that contains the target, such as its type or its assembly, is `Multicast`, and one on another declaration, such as the overridden member in a base type, is `Inherited`.

## Listing and filtering

`list <collection>` lists the objects of a collection that satisfy every `--where` condition:

| Collection | Content |
|---|---|
| `aspect-classes` | The aspect classes |
| `aspect-instances` | The aspect instances: one aspect on one target declaration |
| `predecessors` | What created the aspect instances: a custom attribute, a multicast attribute, an inherited attribute, an XML element, or an aspect provider (kind `ChildAspect`) |
| `transformations` | The changes that the advices make to declarations, in order |
| `annotations` | Information about declarations that is not a transformation, such as the dependencies between properties of `[NotifyPropertyChanged]` |
| `declaration-mappings` | The replacements of source declarations by woven ones, such as a field encapsulated as a property |
| `declarations` | The declarations that the files mention, with their location |
| `files` | The files that the commands read, one per project and target framework, with their write time and whether they are stale |

A condition is an OData 4.01 `$filter` expression, parsed by [Microsoft.OData.Core](https://www.nuget.org/packages/Microsoft.OData.Core), so an agent writes OData as it knows it:

```powershell
postsharp introspect list aspect-instances --where "aspectClass/shortName eq 'Log'" --count
postsharp introspect list declarations --where "aspectInstances/any(a: a/aspectClass/shortName eq 'Cache')" --select "id,location"
postsharp introspect list aspect-instances --where "startswith(targetDeclaration/namespace, 'App.Orders')" --order-by "targetDeclaration/id"
postsharp introspect list aspect-instances --where "'premium' in tags"
postsharp introspect list aspect-instances --where 'transformations/$count($filter=semantic eq ''Getter'') gt 0'
```

The commands evaluate:

* the comparison operators `eq`, `ne`, `gt`, `ge`, `lt` and `le`, the logical operators `and`, `or` and `not`, and `in`, with a list of values or with a collection field;
* the arithmetic operators `add`, `sub`, `mul`, `div`, `mod` and the unary `-`;
* `any` and `all`, with or without a lambda, and `$count` of a collection, with or without `$filter`;
* the functions `contains`, `startswith`, `endswith`, `indexof`, `length`, `substring`, `concat`, `tolower`, `toupper`, `trim` and `matchesPattern`, whose pattern is a .NET regular expression.

Another function, such as a date or a math function, is an invalid query whose message lists the supported functions. The values follow these rules:

* Field names are case-sensitive. A name in the wrong case is an error that suggests the right one.
* Strings are compared ordinally. A value that the file does not record is `null`: an ordering comparison with `null` is false, and `eq null` tests for it.
* Integers are 64-bit. An integer operation that overflows is an invalid query; a division or a modulo by zero is `null`.
* A field or a collection that PostSharp does not record is an error, never an empty result. `schema` says which ones PostSharp records.
* Bash and PowerShell expand `$` inside double quotes: write `$count` in single quotes, as above, or escape the `$`. When the shell removed it, the error says so.
* A condition that starts with `-` is read as an option: write `(-order) lt 0`, or `"--where=-order lt 0"`.

`--select` chooses the fields, as paths separated by commas, such as `aspectClass/shortName,transformations/$count`; an object and one of its fields cannot both be selected. `--order-by` takes OData `$orderby` expressions, such as `length(shortName) desc, shortName`, on fields that are not objects; a `null` value comes first. `--skip` and `--limit` give a page, and `--count` returns only the number of objects. `postsharp introspect schema` describes every collection and field, and says which ones PostSharp records.

## Drilling down from a summary to the details

The output is limited and made of summary rows, so that an agent can read it. To learn more about an object of a result, query it by its identifier, one level at a time, instead of asking for everything at once. The examples below come from a solution in which a `Log` aspect is multicast from an assembly.

1. **Start from an overview.** `list aspect-classes` gives the aspect classes and the number of their aspect instances, and `metrics` the lines of code they save:

   ```powershell
   postsharp introspect list aspect-classes --select 'shortName,name,aspectInstances/$count'
   ```

2. **Show one object.** `show` writes the object with its related objects as summary rows: the identifier, the aspect class, the target declaration and the project of each aspect instance, for instance. `show declaration` is the exception: it writes the aspect instances of the declaration with their predecessors and transformations.

   ```powershell
   postsharp introspect show aspect-class Log --limit 2
   ```

3. **Read the truncated lists before widening them.** A list of related objects says how many objects it has (`count`), whether some were left out (`truncated`), in which namespaces they are (`byNamespace`), and which command lists all of them (`next`):

   ```json
   "aspectInstances": {
     "count": 6,
     "truncated": true,
     "items": [ ... ],
     "byNamespace": [
       { "namespace": "App.Orders", "count": 3 },
       { "namespace": "Lib", "count": 2 },
       { "namespace": "App", "count": 1 }
     ],
     "next": "list aspect-instances --where \"aspectClass/name eq 'Lib.LogAttribute'\" --all"
   }
   ```

   Run `next` with `postsharp introspect` to list all the objects, or narrow it first with another condition, for instance on the namespace that `byNamespace` names:

   ```powershell
   postsharp introspect list aspect-instances --where "aspectClass/name eq 'Lib.LogAttribute' and targetDeclaration/namespace eq 'App.Orders'" --select "id,targetDeclaration/id"
   ```

4. **Show an object of a row by its identifier.** Any unique prefix of an identifier is accepted. `show aspect-instance` gives the predecessors of the aspect instance, which say what created it, and its transformations in order:

   ```powershell
   postsharp introspect show aspect-instance f503
   ```

   ```text
   Aspect Log [f50384e0ebfe] on method App.Orders.Repository.Save(App.Customer).
     Created by the multicast attribute [0374f134812d] on App (kind inferred).
     Transformations, in order:
       2. App.Orders.Repository.Save(App.Customer): Wrapped by advice(s) OnEntry
   ```

5. **Follow a predecessor to everything it created.** `show predecessor` lists the aspect instances that one attribute, multicast attribute or XML element created, which tells the reach of a multicast filter:

   ```powershell
   postsharp introspect show predecessor 0374
   ```

   A predecessor of kind `ChildAspect` names the aspect instance of the aspect provider in its `aspectInstance` field: show it in turn to see what the provider was applied to.

6. **Go back to the source.** `show declaration` takes the identifier or the C# name of a target declaration, and its `location` field gives the position of its name in the source; `list declarations --select id,location` gives the positions of several declarations.

The identifiers stay the same across builds as long as the source does not change, so an identifier found in one call can be used in the next. To ask several of these questions in one call, give them to `batch` (see [below](#running-several-queries-at-once)).

## Running several queries at once

Most of the cost of a command is the evaluation of the projects and the loading of their files. `batch` runs several commands over one load, which is several times faster than separate calls. It reads one command per line, without `introspect`, from the standard input or from `--file`:

```bash
postsharp introspect batch --solution App.sln <<'EOF'
show declaration src/Customer.cs:18
list aspect-instances --where "aspectClass/shortName eq 'Log'" --count
metrics
EOF
```

In PowerShell, pipe a here-string: `@'...'@ | postsharp introspect batch`. The scope and the format are those of the batch. Every query runs, even after one fails, and the JSON output gives the result or the error of each.

A line is split into words as a POSIX shell splits them: `'...'` is literal, and inside `"..."`, a backslash escapes `"`, `\`, `$` and `` ` ``. Outside quotes, a backslash escapes a space, a quote or a backslash, as in `My\ File.cs`; any other backslash is kept, so that a Windows path such as `src\Customer.cs:18` needs no quotes. A line that starts with `#` is a comment. A line cannot set `--project`, `--solution`, `-f`, `-c`, `--format` or `--color`, nor run `batch`; `--help` on a line gives the help of its command as its result. The `next` commands of truncated lists are quoted so that they can be pasted into bash or into a line of a batch.

## Metrics

`metrics` shows the line counts of the Metrics window of PostSharp Tools for Visual Studio, per project and target framework: the lines of code written by hand, the lines saved by aspects (the lines of code avoided that the aspects declare, see <xref:PostSharp.Aspects.LinesOfCodeAvoidedAttribute>), and their ratios.

```powershell
postsharp introspect metrics --solution App.sln
```

| Field | Meaning |
|---|---|
| `linesOfCode` | The lines of code written by hand. |
| `linesSaved` | The lines of code that the aspects avoided: the sum of the estimates of the aspect instances, transformations and annotations. See <xref:estimating-code-saving>. |
| `avoidedRatio` | `linesSaved / linesOfCode`: the code that would have been written by hand without aspects, relative to the code actually written. |
| `savedRatio` | `linesSaved / (linesOfCode + linesSaved)`: the share of the whole code that the aspects produce. |
| `enhancedTypes` | The number of types that aspects changed. |

A ratio is `null` when the project has no hand-written line or no saved line. The `total` counts the first target framework of each project, because the target frameworks of a project share most of their code.

## Output for AI agents

* The output is one JSON document by default; `--format jsonl` writes one object per line, and `--format text` writes text for a person.
* The results do not list the files they come from. `list files` gives them, with their write time and whether they are stale. A stale or missing file is also reported as a warning on the standard error.
* **The output is limited**, so that an agent can read it:
  * `list` writes at most 100 objects, except with `--format jsonl`. Its `count` is the number of objects that satisfy the conditions, whatever the page, and `truncated` says whether objects were left out, with `byNamespace`, the number of objects of each namespace. Use `--skip` and `--limit` for another page, `--count` for the number only, or `--all`.
  * `show` writes at most 20 objects in each list of related objects, by default. Each list is an object with the same shape, whatever its length: `count`, `truncated` and `items`, and, when it is truncated, `byNamespace` and `next`, the command that lists all its objects. Run `next` with `postsharp introspect`, or use `--limit <n>` or `--all`.
  * Except in `show declaration`, the related objects of `show` are summary rows: run `show` on one of them for its details. See [Drilling down from a summary to the details](#drilling-down-from-a-summary-to-the-details).
* The exit code is 0 when the query runs, including when its result is empty; 2 when a file is missing, with a message that says what to build; 3 when a selector is ambiguous; and 4 when a query is invalid or asks for data that PostSharp does not record.
* Colors are used only on a terminal, never when an agent captures the output: the JSON is colored, errors are red and warnings yellow. `NO_COLOR` or `--color never` turns them off.

## Limitations

* The files do not record the messages of the build, nor the aspect instances that were skipped. Read the build output for the messages.
* The weaving symbol file does not record how a predecessor created its aspect instances. The commands infer it, and say so (`isKindInferred`).
* A build with `PostSharpWriteWeavingSymbols` set to `False` writes no weaving symbol file.

## See Also

**Reference**

<xref:PostSharp.Aspects.LinesOfCodeAvoidedAttribute>
<br>

**Other Resources**

<xref:testing-application>
<br><xref:installation>
<br><xref:customize-vs-appearance>
<br><xref:estimating-code-saving>
<br><xref:ai-agents>
<br><xref:msbuild-properties>
<br>
