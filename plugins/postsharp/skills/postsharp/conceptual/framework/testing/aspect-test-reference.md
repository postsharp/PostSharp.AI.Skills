---
uid: aspect-test-reference
title: "Aspect Test Directives and Settings"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Reference of the test directives, MSBuild properties, postsharpTests.json settings and environment variables of the PostSharp aspect test framework."
---
# Aspect Test Directives and Settings

This article lists the settings of the aspect test framework of the `PostSharp.Testing.AspectTesting` package. For an introduction, see <xref:aspect-tests>.

## Test directives

A directive is a comment of a test file that starts with `@` and the name of the directive: `// @SkipRuntime` in C#, `' @SkipRuntime` in Visual Basic. A directive with an argument is written `@Name(argument)`.

* By convention, put the directives at the top of the file. The test framework reads them anywhere in the file, including in string literals.
* A comment that starts with an unknown directive fails the test.
* An argument cannot contain a closing parenthesis.
* When a directive that takes a single value appears several times, the last one applies. The directives that accept several values, such as `@Include` or `@DefineConstants`, accumulate them.
* A directive that follows `@TestRun(Name)` on the same line, prefixed with `=` instead of `@`, applies only to that run, in addition to the directives of the file.
* The `=` must follow a space, for example `// @TestRun(Name) =SkipRuntime =DefineConstants(FOO)`.

### Runs

| Directive | Effect |
|---|---|
| `@TestRun(Name)` | Adds a run of the test, reported as `TestName(Name)`. The name of the run is defined as a preprocessor symbol, and passed to `Main` as its only argument. The expected files of the run are named `TestName.Name` followed by the suffix of the expected file, such as `TestName.Name.txt`. |
The following examples show runs with directives of their own:

<!-- not-compiled: directive lines -->
```csharp
// A run without optimizations, and a run with optimizations.
// @TestRun(Debug)
// @TestRun(Release) =Optimize(true)

// A run that defines a symbol, which the code of the test tests with #if, and a run without the library of the test.
// @TestRun(WithFeature) =DefineConstants(FEATURE)
// @TestRun(WithoutLibrary) =SkipDependency =DefineConstants(NO_LIBRARY)

// A run that is only compiled and verified, and a run that is skipped on Linux and macOS.
// @TestRun(BuildOnly) =SkipRuntime
// @TestRun(WindowsOnly) =SkipOS(Unix)

// A run with another language version and the nullable context disabled.
// @TestRun(CSharp12) =LangVersion(12) =Nullable(disable)
```

### Dependencies

| Directive | Effect |
|---|---|
| `@SkipDependency` | Does not compile the library of the test, `TestName.Dependency.cs` or `TestName.Dependency.vb`. |
| `@SkipPostSharpOnDependency` | Compiles the library of the test without weaving it. |
| `@Reference(Name)` | References `Name.dll`, from the directory of the test. |
| `@SearchPath("path")` | Adds a directory to the search path of PostSharp, where PostSharp looks for plug-ins, such as weavers, and for referenced assemblies, like the `PostSharpSearchPath` property. The path must be quoted. |

### Skipping tests

A skipped test is reported as skipped by the test runner, with the reason.

| Directive | Effect |
|---|---|
| `@Ignore(reason)` | Skips the test. The reason is optional. |
| `@SkipOS(Windows)` | Skips the test on an operating system: `Windows`, `Unix` (any system but Windows), `Linux` or `MacOS`. |
| `@ExcludeFramework(NET)` | Skips the test on a family of target frameworks: `NET` for .NET Framework, `NETCore` for .NET and .NET Core. |
| `@MinFrameworkVersion(NETCore,8.0)`, `@MaxFrameworkVersion(NETCore,8.0)` | Skips the test when the target framework of the test application is of that family and below or above the version. |
| `@MinToolsVersion(17.0)`, `@MaxToolsVersion(17.0)` | Skips the test when the version of MSBuild that built the test project is below or above the version. |
| `@LangVersion(14)` | Sets the C# or Visual Basic language version. Skips the test when the compiler does not know the version. |
| `@UseWpf(true)`, `@UseWindowsForms(true)` | Overrides `PostSharpTestUseWpf` or `PostSharpTestUseWindowsForms` for this test. A test is discovered only by a test project whose `UseWPF` and `UseWindowsForms` properties both equal the effective values of the test. The other tests are not reported. |

### Compilation

| Directive | Effect |
|---|---|
| `@Include(file)` | Compiles another file with the test. The path is relative to the directory of the test. |
| `@DefineConstants(A;B)` | Defines preprocessor symbols, separated by `;` or `,`. |
| `@Nullable(disable)` | Sets the nullable context of a C# test, instead of the `Nullable` property of the test project. |
| `@Optimize(true)` | Compiles the test with optimizations. The default is `false`. The argument is `true`, `on`, `yes`, `false`, `off` or `no`, in any case. Any other value fails the test. |
| `@DebugSymbols(false)` | Compiles the test without a PDB. The default is `true`. |

### PostSharp

| Directive | Effect |
|---|---|
| `@ExpectedMessage(Code)` | Expects a message from PostSharp or from an aspect, when the test project does not set `PostSharpTestMessagesFileSuffix`. Ignored otherwise: the expected messages file applies. |
| `@DisabledMessages(PS0131)` | Disables messages, like the `PostSharpDisabledMessages` property. The codes are separated by `;`, `,` or spaces. They also apply to the library of the test. |
| `@Property(Name=Value)` | Sets a property of the PostSharp compiler. |
| `@BinaryFormatterAllowed` | Allows aspects to be serialized with `BinaryFormatter`. |

### Verification

| Directive | Effect |
|---|---|
| `@SkipILVerify` | Does not verify the woven assembly with ILVerify. |
| `@IgnoreILVerify(Code)` | Ignores an error code of ILVerify. The test fails when no error needs the ignore, unless `@SuppressILVerifyUnusedIgnore` names the code. |
| `@SuppressILVerifyUnusedIgnore(Code)` | Does not fail the test when `@IgnoreILVerify(Code)` is not needed. |
| `@RequiredSymbol(text)`, `@ForbiddenSymbol(text)` | Checks that the symbol file of PostSharp (`.pssym`) contains, or does not contain, the text. |

### Execution

| Directive | Effect |
|---|---|
| `@SkipRuntime` | Compiles, weaves and verifies the test, but does not run it. |
| `@IgnoreOutput` | Runs the program, which must return 0, but does not compare its output. |
| `@CopyToOutput(file)` | Copies a file of the directory of the test next to the program, which is also its current directory. |
| `@AppConfig(file)` | Gives the program a configuration file, and runs it in a process of its own. Not supported on .NET Framework, where the program receives the configuration file of the test application. |
| `@RunOutOfProcess` | Runs the program in a process of its own on .NET, for instance when it depends on the name of the entry assembly. On .NET Framework, every program runs in a process of its own. |

## MSBuild properties

Set these properties in the test project:

| Property | Default | Description |
|---|---|---|
| `PostSharpTestSourceDirectory` | The project directory | The directory whose source files are the tests. |
| `PostSharpTestOutputFileSuffix` | Empty | The suffix of the expected output files, such as `.txt` for `Foo.txt`. When empty, the output of the programs is not compared. |
| `PostSharpTestMessagesFileSuffix` | Empty | The suffix of the expected messages files, such as `.build.txt` for `Foo.build.txt`. When empty, the PostSharp compiler checks the `@ExpectedMessage` directives and fails on unexpected messages. |
| `PostSharpTestAcceptResults` | `False` | `True` overwrites the expected files that differ with the actual results, instead of comparing them, and deletes the expected files whose actual result is empty. The build records the property: the test application accepts the actual results until it is built again without the property. |
| `PostSharpTestAssemblyNameFromFileName` | `False` | `True` names the assembly of each test after its file, `Foo` for `Foo.cs`, instead of after its directory and file, `Directory.Subdirectory-Foo`. In both cases, a run appends `-Run` to the name. |
| `PostSharpTestDisplayNameSuffix` | Empty | A suffix added to the display name of each test, between brackets, to tell apart the results of several configurations of the same tests, such as `MyTest [Release]`. The build records the property. |
| `PostSharpTestUseWpf`, `PostSharpTestUseWindowsForms` | `False` | Whether the tests use WPF or Windows Forms, unless `@UseWpf` or `@UseWindowsForms` says otherwise. Set it to `True` in a test project that sets `UseWPF` or `UseWindowsForms`, or that project discovers only the tests marked `@UseWpf(true)` or `@UseWindowsForms(true)`. |
| `PostSharpTestVisualBasicProperties` | Empty | Additional properties of the Visual Basic project from which the compiler options of the `.vb` tests are taken, for example `RootNamespace=MyTests`. |
| `PostSharpTestPublishMode` | Empty | On .NET, publishes each woven program and runs the published program. The value is `ReadyToRun`, `TrimmedReadyToRun` or `NativeAot`. With `NativeAot`, the tests are woven with `PostSharpAotCompatibleWeaving`. |
| `PostSharpTestAotCompatibleWeaving` | `True` | `False` weaves the tests without `PostSharpAotCompatibleWeaving` when `PostSharpTestPublishMode` is `NativeAot`. |
| `PostSharpTestWeavingMode` | `InProcess` on .NET | `InProcess` weaves the tests in the test process, several at a time. `OutOfProcess` starts a compiler process for each test, as a build does. On .NET Framework, the tests always weave out of process. |
| `PostSharpTestCheckUnloading` | `False` | `True` adds the test `<Project>.InProcessCompilations.AreUnloaded`, which runs after the other tests, only when the tests weave in process. It runs full garbage collections until no compilation is alive, and fails after 30 seconds. See <xref:in-process-memory-leaks>. |
| `PostSharpTestDumpOnLeak` | `False` | `True` writes a memory dump of the test process when `AreUnloaded` fails, to `TestResults/leak-dumps`, and, when the tests run on TeamCity, publishes it as a build artifact. See <xref:in-process-memory-leaks>. |

The following properties of the test project also apply to every test:

* For C# tests, `Nullable`, `NoWarn`, `WarningLevel` and `DefineConstants`. The `LangVersion` property does not apply: C# tests are compiled with the latest language version of the compiler, unless they have the `@LangVersion` directive.
* `TreatWarningsAsErrors` and `WarningsAsErrors`, for the messages of PostSharp only.
* The PostSharp properties, such as `PostSharpDisabledMessages`, `PostSharpEscalatedMessages`, `PostSharpCompatibilityLevel` or `PostSharpConstraintVerificationEnabled` (see <xref:configuration-msbuild>).

## MSBuild targets

| Target | Description |
|---|---|
| `AcceptTestOutput` | Copies the actual results that the failed tests wrote, such as `Foo.actual.txt` and `Foo.build.actual.txt`, over their expected files, and deletes the actual files. An empty actual result deletes the expected file. The target does not build the project or run the tests: run it with `dotnet build -t:AcceptTestOutput` after the tests. |

## The postsharpTests.json file

A `postsharpTests.json` file configures the tests of its directory and of its subdirectories. It can contain comments.

| Setting | Description |
|---|---|
| `Exclude` | `true` excludes the directory and its subdirectories from test discovery. |
| `IncludedFiles` | Files to compile with every test of the directory, relative to the `postsharpTests.json` file. A test compiles only the files of its own language. The included files are not tests. |
| `IsRoot` | `true` ignores the `postsharpTests.json` files of the parent directories. |

Without `IsRoot`, the settings of the parent directories apply, up to `PostSharpTestSourceDirectory`, and the included files of the parent directories come first.

## Environment variables

| Variable | Description |
|---|---|
| `DebugPostSharpTestFramework` | Setting the variable to any value launches a debugger when the test application starts. |
| `LogPostSharpTestFramework` | Setting the variable to any value reports the trace of test discovery as diagnostic messages. |
| `POSTSHARP_TEST_HOLD_AT_END` | A number of minutes for which the test application waits after the last test, after full garbage collections, so that a memory dump can be taken. See <xref:in-process-memory-leaks>. |
| `POSTSHARP_TEST_MEMORY_LOG` | The path of the CSV file of memory usage that the test framework writes after each weaving in process, instead of `TestResults/inprocess-memory.csv`. See <xref:in-process-memory-leaks>. |
| `POSTSHARP_MAX_PARALLELISM` | The number of tests that weave in process at a time. The default is half the processor count. See <xref:in-process-memory-leaks>. |

## See Also

**Other Resources**

<xref:aspect-tests>
<br><xref:in-process-memory-leaks>
<br><xref:testing-aspects>
<br>[PostSharp.Testing.AspectTesting on NuGet](https://www.nuget.org/packages/PostSharp.Testing.AspectTesting/)
