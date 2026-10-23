---
uid: aspect-tests
title: "Testing Aspects with the Aspect Test Framework"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Test aspects with the PostSharp.Testing.AspectTesting package. Each source file of a test project is compiled, woven, verified and run as a test of its own."
---
# Testing Aspects with the Aspect Test Framework

The `PostSharp.Testing.AspectTesting` package is a test framework for aspects. Each source file of a test project is a small program to which you apply your aspects. For each file, the test framework compiles the program, weaves it with PostSharp, verifies the woven code with ILVerify, and runs the program. The test passes when the build reports no unexpected error or warning, and when the program completes successfully.

Aspect tests cover cases that ordinary unit tests of aspects cannot easily cover. An ordinary test project is compiled and woven as a whole. If an aspect emits a build error, the build of every test fails, and an error cannot be the expected result of a test. In an aspect test project, each test is compiled and woven in its own assembly. A test can therefore expect an error, and a failure stays in the test that causes it.

Optionally, the test framework can also compare the output of each program and the messages of its build with *expected files* stored next to the test. The test framework does not compare the woven IL.

To create aspect tests:

1. Create a test project that references `PostSharp.Testing.AspectTesting`.
2. Add a test file: a program that applies the aspect and checks its behavior.
3. Build the project and run the tests.

## Creating a test project

Create a project with the following content:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>

    <!-- The tests are xunit.v3 tests that run on Microsoft.Testing.Platform. -->
    <UseMicrosoftTestingPlatformRunner>true</UseMicrosoftTestingPlatformRunner>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="xunit.v3" Version="..." />
    <PackageReference Include="PostSharp.Testing.AspectTesting" Version="..." />

    <!-- The aspects under test. -->
    <ProjectReference Include="..\MyAspects\MyAspects.csproj" />
  </ItemGroup>

</Project>
```

The test project must target .NET 10 or later, or .NET Framework 4.7.2 or later. Tests that target .NET Framework run only on Windows. To test your aspects on several target frameworks, set `TargetFrameworks` instead of `TargetFramework` (see [Testing several target frameworks](#testing-several-target-frameworks)).

Reference `PostSharp.Testing.AspectTesting` directly from the test project. A package that is referenced only transitively does not apply its MSBuild files. The package depends on the `PostSharp` package of the same version, so referencing `PostSharp` is optional. If you reference it, use the same version as `PostSharp.Testing.AspectTesting`.

The package changes the test project as follows:

* The `.cs` and `.vb` files of the project directory are not compiled into the test project. Each one is a test, which the test framework compiles separately.
* The test project itself is not woven: the package sets `SkipPostSharp`. The `PostSharp` package is used only to compute the inputs of the PostSharp compiler. Each test uses these inputs.
* The project is an executable, and its tests are run by the test framework of the package instead of by test methods.

> [!WARNING]
> Do not add `PostSharp.Testing.AspectTesting` to a project that contains other code or other tests. Because the package removes the source files of the project from the compilation, use a project that is dedicated to aspect tests.

A test is compiled against the references of the test project. A test can therefore use only the packages and projects that the test project references. When some tests need other packages, put them in another test project. A test cannot use xunit, Roslyn or MSBuild, even when the test project references them.

For instance, to test code that uses the PostSharp Pattern Libraries, such as caching or logging, add the `PostSharp.Patterns.Caching` or `PostSharp.Patterns.Diagnostics` package to the test project, as you would in any other project. The aspects of these packages are then woven into each test that uses them.

For the other properties of the test project, see <xref:aspect-test-reference>.

## Adding a test

A test is a source file that is a complete program: it declares a `Main` method, and the test framework runs the entry point of the compiled assembly. The program checks the behavior of the aspect itself. It fails the test by throwing an exception or by returning an exit code other than 0.

Suppose that you want to test the following aspect, which rejects negative arguments:

[!code-csharp[](../../../code/samples/CustomPatterns/Testing/AspectTests/AssertionTest.cs#aspect)]

To test this aspect, create a file named `AssertionTest.cs` in the test project. Apply the aspect to a method, call this method from `Main`, and check the results:

[!code-csharp[](../../../code/samples/CustomPatterns/Testing/AspectTests/AssertionTest.cs#test)]

In a real test project, the aspect is declared in the referenced project. The samples in this article declare it in the test file for brevity.

The test framework discovers every `.cs` and `.vb` file in the directory of the test project and in its subdirectories, except:

* files whose name starts with an underscore (`_`), which you can share between tests with the `@Include` directive;
* files whose name contains `.Dependency.`, which are the libraries of other tests (see [Testing an aspect across assemblies](#testing-an-aspect-across-assemblies));
* the `bin` and `obj` directories, and the directories that a `postsharpTests.json` file excludes (see [Configuring tests by directory](#configuring-tests-by-directory)).

Each test is compiled alone, so implicit usings do not apply to the tests. Write every `using` directive.

## Running the tests

Build the test project before running the tests. The build writes the inputs of the PostSharp compiler next to the test application, and the test framework reads them. When you change your aspects, the test project or a package version, build the test project again. The test files and the `postsharpTests.json` files are read when the tests run, so editing a test requires no build.

Then run the tests like other xunit.v3 tests. Use the test explorer of your IDE, or run the test application, which accepts the options of Microsoft.Testing.Platform:

```powershell
# Build the test project and run one test.
dotnet run -- --filter-method "*.AssertionTest"

# Run the tests of one directory.
dotnet run -- --filter-class "MyAspects.Tests.Validation"
```

To run the tests with `dotnet test` on the .NET 10 SDK or later, opt in to the Microsoft.Testing.Platform mode of `dotnet test` in the `global.json` file of the repository:

```json
{
  "test": {
    "runner": "Microsoft.Testing.Platform"
  }
}
```

Each file is a test method named after the file without its extension. A Visual Basic test keeps its `.vb` extension. The test class is named after the directory of the test project, followed by the relative path of the directory of the test, with dots: the tests of `Validation/` in the `MyAspects.Tests` directory are in the `MyAspects.Tests.Validation` class. The tests at the root, and the tests of a directory that also has subdirectories, are in a class whose name ends with `.Tests`.

The tests run in parallel, up to twice the number of processors.

For each test, the test framework performs the following steps:

1. It compiles the test file with Roslyn, with the references, the nullable context, the defined constants and the suppressed warnings of the test project.
2. It weaves the compiled assembly with the PostSharp compiler, as a build would.
3. It checks the errors and warnings of the compilation and of the weaving. When the build has an error, the test stops here.
4. It verifies the woven assembly with ILVerify.
5. It runs the program, which must return 0 or `void` without throwing an exception. When a previous step failed, the program is not run.

A test is compiled without optimizations and with a PDB, whatever the build configuration of the test project, and with the `DEBUG`, `TRACE` and `POSTSHARP` constants. C# tests are compiled with the latest language version of the compiler, whatever the `LangVersion` property of the test project. Use the `@LangVersion`, `@Optimize` and `@DebugSymbols` directives to change these options.

The test writes its intermediate files to `obj/aspect-tests/<target framework>/` under the test project, in a directory for each test. The test output, which your test runner displays, gives the name of this directory, the messages of the compiler and of PostSharp, and the output of the program.

On .NET, the program runs in the process of the test framework, in a collectible <xref:System.Runtime.Loader.AssemblyLoadContext> of its own, so the assemblies of each test are loaded in isolation from the other tests. On .NET Framework, each program runs in a process of its own.

Because the programs share the process of the test framework on .NET, they run one at a time, and they share its environment variables and its entry assembly. A program that calls `Environment.Exit` ends the test application. Add the `@RunOutOfProcess` directive to a test that depends on the process.

## Testing errors and warnings

An aspect can validate its target at build time and emit an error when it is misused (see <xref:aspect-validation>). A test can check these errors and warnings in two ways:

* With the `@ExpectedMessage` directive, which declares each message that the test expects. This is the default.
* With an expected messages file, such as `Foo.build.txt`, which lists all the errors and warnings of the build of the test, with their text and their location (see [Comparing errors and warnings with an expected file](#comparing-errors-and-warnings-with-an-expected-file)).

By default, the PostSharp compiler checks the messages of each test: any error or warning of PostSharp or of an aspect fails the test, unless the test expects it with the `@ExpectedMessage` directive:

<!-- not-compiled: a directive line, not a program -->
```csharp
// @ExpectedMessage(MY001)
```

A test that applies no aspect gets the PS0131 warning, because its assembly does not need PostSharp. Disable it with `@DisabledMessages(PS0131)`.

A test that expects a message passes when PostSharp reports it. When the expected message is an error, the program is not run. An error of the C# or Visual Basic compiler always fails the test.

To check the text and the location of the messages, and the warnings of the C# compiler, use an expected messages file instead.

## Comparing results with expected files

Instead of checking its results in code, a test can compare two results with expected files stored next to it: the console output of its program, and the errors and warnings of its build. Expected files are optional. You enable them for the whole test project with two properties:

```xml
<PropertyGroup>
  <!-- Compare the console output of each program with Foo.txt. -->
  <PostSharpTestOutputFileSuffix>.txt</PostSharpTestOutputFileSuffix>

  <!-- Compare the errors and warnings of the build of each test with Foo.build.txt. -->
  <PostSharpTestMessagesFileSuffix>.build.txt</PostSharpTestMessagesFileSuffix>
</PropertyGroup>
```

Each property is independent. A missing expected file stands for an empty result: a program that writes nothing to the console needs no output file, and a test whose build reports no error or warning needs no messages file.

### Comparing the console output with an expected file

With `PostSharpTestOutputFileSuffix`, the test framework compares the console output of each program with the expected output. The following aspect logs the calls of a method:

[!code-csharp[](../../../code/samples/CustomPatterns/Testing/AspectTests/LogAspectTest.cs#aspect)]

The test `LogAspectTest.cs` calls a method to which the aspect is applied:

[!code-csharp[](../../../code/samples/CustomPatterns/Testing/AspectTests/LogAspectTest.cs#test)]

The expected output, in `LogAspectTest.txt`, is:

[!code-text[](../../../code/samples/CustomPatterns/Testing/AspectTests/LogAspectTest.txt)]

To run a test whose output varies, for instance because it contains durations, without comparing its output, add the `@IgnoreOutput` directive to the test.

### Comparing errors and warnings with an expected file

With `PostSharpTestMessagesFileSuffix`, the test framework records the errors and warnings of the compilation and of the weaving, and compares them with the expected messages of the test. The `@ExpectedMessage` directives are then ignored.

The following aspect emits an error when it is applied to a method that returns `void`:

[!code-csharp[](../../../code/samples/CustomPatterns/Testing/AspectTests/ValidationErrorTest.cs#aspect)]

To test this error, apply the aspect to such a method:

[!code-csharp[](../../../code/samples/CustomPatterns/Testing/AspectTests/ValidationErrorTest.cs#test)]

The expected messages, in `ValidationErrorTest.build.txt`, contain the error:

[!code-text[](../../../code/samples/CustomPatterns/Testing/AspectTests/ValidationErrorTest.build.txt)]

The expected messages file has one message per line, in the form `<severity> <code>: <file>(<line>,<column>): <text>`, sorted by location. The path of the file is relative to the directory of the test.

* The file contains the warnings and errors of the C# or Visual Basic compiler, of PostSharp and of your aspects.
* It does not contain warnings without a code, or the NuGet, .NET SDK and MSBuild messages (`NU`, `NETSDK` and `MSB` codes).
* The .NET analyzers do not run.

When the build has an error, the program is not run. A test that expects an error also fails if the build reports a different error, so a failure for an unrelated reason cannot go unnoticed.

### Accepting the actual results

When a result differs from its expected file, the test fails and writes the actual result next to the expected file, with `.actual` inserted before the last extension: `LogAspectTest.actual.txt` for the console output, and `ValidationErrorTest.build.actual.txt` for the errors and warnings. A new test whose program writes to the console, or whose build reports errors or warnings, therefore fails once, and writes its first actual result.

Review the actual results. When they are correct, run the `AcceptTestOutput` target of the test project. The target copies each actual file over its expected file, and deletes the actual file. It does not build the project or run the tests:

```powershell
# Run the tests. The failed tests write their actual results.
dotnet run

# Accept the actual results of the failed tests.
dotnet build -t:AcceptTestOutput

# Review the new expected files.
git diff
```

When an actual result is empty, accepting it deletes the expected file, because a missing expected file stands for an empty result.

Alternatively, set the `PostSharpTestAcceptResults` property to `True` when you build the test project. The tests then overwrite the expected files that differ with the actual results instead of comparing them, and pass. The build records the property, so the test application keeps accepting the actual results until you build it again without the property:

```powershell
# Commit or stage your changes first, so that you can review what the tests change.
dotnet run -p:PostSharpTestAcceptResults=True

# Build the test project again, so that the next runs compare the results.
dotnet build
```

The comparison ignores line endings, trailing whitespace, and leading and trailing blank lines. The paths of the test directory are removed from the output, so the expected files do not depend on the location of the repository.

A `.actual` file is deleted when its test passes. Add `*.actual.*` to your `.gitignore` file, so that the files of failed tests are not committed.

## Using test directives

A test configures itself with *directives*: comments that start with `@` followed by the name of the directive, such as `// @SkipRuntime` in C# or `' @SkipRuntime` in Visual Basic. Put them at the top of the file. A comment that starts with an unknown directive fails the test.

The most common directives are:

| Directive | Effect |
|---|---|
| `@SkipRuntime` | Compiles, weaves and verifies the test, but does not run it. |
| `@Ignore(reason)` | Skips the test. The test runner reports it as skipped, with the reason. |
| `@Include(file)` | Compiles another file of the test directory with the test. |
| `@DefineConstants(A;B)` | Defines preprocessor symbols. |
| `@Nullable(disable)` | Sets the nullable context of the test. |
| `@DisabledMessages(PS0131)` | Disables PostSharp messages, like the `PostSharpDisabledMessages` property. |
| `@CopyToOutput(file)` | Copies a file of the test directory next to the program, which is also its current directory. |
| `@RunOutOfProcess` | Runs the program in a process of its own on .NET. |

For the complete list, see <xref:aspect-test-reference>.

## Running a test several times

The `@TestRun(Name)` directive adds a *run* of the test. Each run is a separate test case, which the test runner shows as `TestName(Name)`. Use runs to test the same code with different directives, for instance with and without optimizations.

A run gets the directives of the file, and the directives that follow its name on the same line, each one prefixed with `=` instead of `@`. For instance, the following test has three runs:

<!-- not-compiled: the directive lines of the sample below -->
```csharp
// @TestRun(Debug)
// @TestRun(Release) =Optimize(true)
// @TestRun(Disabled) =DefineConstants(DISABLE_ASPECT) =DisabledMessages(PS0131)
```

* The `Debug` run compiles the test with the directives of the file only.
* The `Release` run also compiles the test with optimizations.
* The `Disabled` run also defines the `DISABLE_ASPECT` symbol, which removes the aspect from the code, and disables the PS0131 warning that a test without aspects gets.

For each run, the test framework also defines the name of the run as a preprocessor symbol, and passes it to `Main` as its only argument:

[!code-csharp[](../../../code/samples/CustomPatterns/Testing/AspectTests/TestRunTest.cs#test)]

The expected files of a run are named after the test and the run, such as `TestRunTest.Release.txt`. The expected output of the `Release` run shows that the aspect also runs in optimized code:

[!code-text[](../../../code/samples/CustomPatterns/Testing/AspectTests/TestRunTest.Release.txt)]

The expected output of the `Disabled` run shows that the method runs without the aspect:

[!code-text[](../../../code/samples/CustomPatterns/Testing/AspectTests/TestRunTest.Disabled.txt)]

## Testing an aspect across assemblies

Some aspects behave differently when they are applied in another assembly, for instance when the aspect is inherited from a base class of a library, or when an architecture constraint applies to references from another assembly. To test them, put the code of the library in a file named after the test followed by `.Dependency.cs`, in the same directory:

```text
InternalImplement/
  InternalImplement.cs              the test program
  InternalImplement.Dependency.cs   the library that the program references
```

The test framework compiles and weaves `InternalImplement.Dependency.cs` as a library, before the test, and references it from the test. The library must be in the language of the test. It is compiled with the constants and the references of the test, and with its own directives. The directives of a run do not apply to it. The `@SkipPostSharpOnDependency` directive compiles the library without weaving it. The `@SkipDependency` directive ignores the library. It is typically used in a single run of the test, as `=SkipDependency`.

## Configuring tests by directory

A `postsharpTests.json` file configures the tests of its directory and of its subdirectories. For instance, the following file compiles `Common.cs` with each test of the directory:

```json
{
  "IncludedFiles": [ "Common.cs" ]
}
```

The paths of the included files are relative to the `postsharpTests.json` file. The included files are not tests themselves. To exclude a directory and its subdirectories from test discovery, set `"Exclude": true`. A `postsharpTests.json` file inherits the settings of the files of the parent directories, up to the directory of the tests (`PostSharpTestSourceDirectory`), unless it sets `"IsRoot": true`.

The PostSharp properties of the test project, such as `PostSharpDisabledMessages` or `PostSharpCompatibilityLevel`, apply to every test. To configure PostSharp for a single test, put the test in a directory of its own with a `postsharp.config` file. The PostSharp compiler reads the `postsharp.config` files of the directory of the test and of its parent directories (see <xref:configuration-system>).

## Testing several target frameworks

A test is compiled for the target framework of the test application that runs it. To test your aspects on several target frameworks, for instance on .NET and on .NET Framework, set `TargetFrameworks` in the test project:

```xml
<PropertyGroup>
  <TargetFrameworks>net10.0;net48</TargetFrameworks>
</PropertyGroup>
```

The build produces a test application for each target framework, and each one runs every test. The test runner shows the results of each target framework separately. The intermediate files of each target framework are in their own directory, `obj/aspect-tests/<target framework>/`.

All the target frameworks share the expected files of a test, so the program must write the same console output on every target framework. To skip a test on some target frameworks, use the `@ExcludeFramework`, `@MinFrameworkVersion` or `@MaxFrameworkVersion` directive.

The tests that target .NET Framework run only on Windows. Each program runs in a process of its own.

## Testing Visual Basic

The test project is a C# project, but it can contain Visual Basic tests: each `.vb` file is a test, which the test framework compiles with the options of a Visual Basic project. To give the Visual Basic tests properties such as `RootNamespace`, set the `PostSharpTestVisualBasicProperties` property of the test project.

## Limitations

* The test files are not compiled into the test project, so the IDE does not report their errors. Run the test to see them.
* A C# test and a Visual Basic test with the same name in the same directory share their expected files.
* Source generators, analyzers and XAML files are not run or compiled.
* The PostSharp compiler runs in a process of its own, as in a build. To debug the build-time logic of an aspect, see <xref:debugging-buildtime>.
* The tests do not test the MSBuild integration of PostSharp, such as restore or incremental builds.

## See Also

**Other Resources**

<xref:aspect-test-reference>
<br><xref:testing-aspects>
<br><xref:simple-tests>
<br><xref:aspect-validation>
<br><xref:debugging-buildtime>
<br>[PostSharp.Testing.AspectTesting on NuGet](https://www.nuget.org/packages/PostSharp.Testing.AspectTesting/)
