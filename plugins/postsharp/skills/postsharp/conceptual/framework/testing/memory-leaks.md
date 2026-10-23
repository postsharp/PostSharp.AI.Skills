---
uid: in-process-memory-leaks
title: "Diagnosing Memory Leaks of In-Process Compilations"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains why build-time code can keep a compilation in memory when several compilations share a process, which rules prevent it, and how to detect a leak and find its root with a memory dump."
---
# Diagnosing Memory Leaks of In-Process Compilations

On .NET, PostSharp can run several compilations in one process: the aspect test framework weaves its tests in the test process, and the .NET pipe server compiles the projects of every build of the current user. Each compilation loads the assemblies of the project, including its aspect libraries, into an assembly load context of its own, and unloads it when the compilation ends. When something that outlives the compilation still references an object, a type or an assembly of that context, the runtime cannot unload it: the whole compilation stays in memory. This article explains how build-time code causes such a leak, how to avoid it, and how to find its root.

## When compilations share a process

| Host | When | Effect of a leak |
|---|---|---|
| The aspect test framework (`PostSharp.Testing.AspectTesting`) | On .NET, by default (`PostSharpTestWeavingMode=InProcess`). See <xref:aspect-tests>. | The memory of the test process grows with each test. With `PostSharpTestCheckUnloading=True`, the run fails. |
| The .NET pipe server | `dotnet build` of a .NET project, by default, except for x86 (`PostSharpUsePipeServer`). | The server uses more memory with each build. It replaces itself when more than 10 compilations stay in memory, which costs a new start. With `PostSharpVerifyLoadContextUnload=True`, the build fails with PS0286. |

A compilation cannot leak when it runs in a process of its own (`dotnet exec`, the .NET Framework compiler) or in an application domain that is unloaded after it (the .NET Framework pipe server).

## Shared assemblies

A *shared assembly* is loaded once per process and never unloaded. The other assemblies of a compilation are loaded into its own context, and unloaded with it.

| Loaded | Assemblies | Static state |
|---|---|---|
| Once per process: shared assemblies | The .NET runtime and its frameworks; `PostSharp.dll` and the compiler; the PostSharp Patterns libraries and weavers whose package has the version of the PostSharp package. | Shared by all the compilations of the process, and alive as long as the process. |
| Per compilation, unloaded after it | The project, its references, and the aspect libraries that it references, including yours. | Each compilation has its own copy, which disappears with the compilation. |

The static fields of your own types are therefore safe: they belong to the load context of the compilation. A leak happens when build-time code stores something of the compilation in the state of a shared assembly.

## How build-time code causes a leak

A leak typically comes from code that runs at build time although it was written for run time, or from process-wide state that an API keeps on behalf of its caller. Build-time code is the code of an aspect that runs during the compilation, such as its constructor, `CompileTimeInitialize`, `CompileTimeValidate` or an `IAspectProvider`, and every static constructor and module initializer that it triggers.

Pay attention to the following cases:

* **Run-time code that runs at build time.** PostSharp instantiates aspects and executes the assemblies of the project at build time, so constructors, static constructors and module initializers run during the compilation. When they prepare the run time, for instance by subscribing to an event, starting a timer or configuring logging, they do it in the process of the compiler. The `[ModuleInitializer]` of a referenced library runs too: a test helper library whose module initializer subscribed to `TaskScheduler.UnobservedTaskException` leaked every compilation of its tests.
* **Static events of the framework.** A handler of `AppDomain.CurrentDomain.AssemblyResolve`, `AppDomain.CurrentDomain.UnhandledException`, `AppDomain.CurrentDomain.ProcessExit`, `TaskScheduler.UnobservedTaskException`, `AssemblyLoadContext.Default.Resolving` or another static event references the type that declares it.
* **Threads, timers and registered waits.** They reference their delegate and capture the execution context of the compilation. This includes the timers and waits that a component of the framework starts the first time it is used: the system proxy, which `HttpClient` creates on its first request, kept the first compilation that sent one until PostSharp started creating it before any compilation.
* **Process-wide registrations.** `AppDomain.CurrentDomain.SetData`, `AppContext.SetData`, `Trace.Listeners` and `Debug.Listeners`, `Console.SetOut` and `Console.SetError`, `TypeDescriptor.AddProvider` and `TypeDescriptor.AddAttributes`, `DiagnosticListener.AllListeners.Subscribe` and `ActivitySource.AddActivityListener` keep what you give them for the life of the process.
* **Caches of shared code keyed by a type.** A static cache of a shared assembly that stores a type of the project, a `MethodInfo` or an instance keeps the compilation's assembly load context. A typical example is cached serializers. PostSharp and its Pattern Libraries keep such caches per compilation; if a dump shows one of their caches, report it as a bug.

## Rules for build-time code

* **Do not subscribe to static events at build time.** Build-time code has no reason to handle an event of the process.
* **Do not start threads, timers or background tasks at build time.** There is no scenario for them in build-time code.
* **Gate run-time logic on <xref:PostSharp.Extensibility.PostSharpEnvironment.IsPostSharpRunning>.** Every class that is instantiated or initialized at build time, which includes aspects, the types that their constructors use, and the static constructors and module initializers of the project and of its references, must skip the logic that prepares the run time when `IsPostSharpRunning` is `true`:

  [!code-csharp[](../../../code/samples/framework/testing/BuildTimeGate.cs#gate)]

* **Do not register anything process-wide at build time**, such as the registrations listed above.
* Keep build-time state in the aspect instance, or in static fields of your own types: they belong to the load context of the compilation.
### Build-time state of a shared assembly

This section applies to the code of a shared assembly, such as a PostSharp Patterns library. Your own aspect libraries are not shared and do not need it.

The code of a shared assembly must not keep build-time state in static fields: every compilation of the process would share it, and it would keep in memory the first compilation that writes it. Store the state in a <xref:PostSharp.Extensibility.ProjectLocal`1>, held in a static field. Each project then has its own value, which is freed with the project:

<!-- not-compiled: ProjectLocal<T> is not in the PostSharp package that the samples reference yet -->
```csharp
private static readonly ProjectLocal<HashSet<MethodInfo>> processedMethods =
    new( "PostSharp.Patterns.Example.ExampleAspect.processedMethods", () => new HashSet<MethodInfo>() );

// At build time:
lock ( processedMethods.Value )
{
    processedMethods.Value.Add( method );
}
```

Use the constructor that takes a slot name, and make the name unique in the process: the full name of the declaring type and the name of the field. Every copy of the assembly then shares one storage slot. The threads of a project share its value, so synchronize the access to a mutable value.

## Detecting a leak

### In the aspect tests

Set these properties in the test project (see <xref:aspect-test-reference>):

| Property | Effect |
|---|---|
| `PostSharpTestCheckUnloading=True` | Adds the test `<Project>.InProcessCompilations.AreUnloaded`, which runs after the other tests, only when the tests weave in process. It runs full garbage collections until no compilation is alive, and fails after 30 seconds with the list of the compilations still alive. Each compilation is named `PostSharp project #<n>` and followed by the assemblies that it loaded, which name the test. |
| `PostSharpTestDumpOnLeak=True` | When that test fails, writes a memory dump of the test process first. See [Capturing a memory dump](#capturing-a-memory-dump). |

After each weaving, the test framework also writes the memory usage of the process and the number of compilations still alive to the CSV file `TestResults/inprocess-memory.csv`. Every 50 weavings, after full garbage collections, it lists the leaked compilations in `TestResults/inprocess-leaks.txt`. Both files are in the output directory of the test application. A number of compilations that grows after full garbage collections shows a leak.

To find the test that leaks, run the test class that the list names, with the check:

```powershell
.\MyTests.exe --filter-class MyTests.Aspects.Caching --filter-class MyTests.InProcessCompilations
```

### In a build

With `PostSharpVerifyLoadContextUnload=True`, the .NET pipe server checks after each compilation that it is unloaded, and reports error PS0286 when the compilation is still in memory 5 seconds later. Set the `POSTSHARP_PIPE_SERVER_LOG` environment variable to a file name to get the log of the server, which lists the compilations it ran. `{pid}` in the name is replaced by the process identifier.

To build without the .NET pipe server while you fix the leak, set `PostSharpUsePipeServer=False`.

## Capturing a memory dump

Use one of these methods:

* **On a build server.** Set `PostSharpTestDumpOnLeak=True`. When `AreUnloaded` fails, the test framework writes a dump with the heap of the test process to `TestResults/leak-dumps/<name>.zip`, and, when the tests run on TeamCity, publishes it as a build artifact. It asks the runtime for the dump through its diagnostics channel, so no tool has to be installed. Diagnostics must not be disabled with `DOTNET_EnableDiagnostics=0`. Extract the dump from the archive before you analyze it.
* **On a development machine.** Set the `POSTSHARP_TEST_HOLD_AT_END` environment variable to a number of minutes. After the last test, the test framework runs full garbage collections, prints the process identifier, and waits. Take the dump while it waits:

  ```powershell
  $env:POSTSHARP_TEST_HOLD_AT_END = '10'
  .\MyTests.exe --filter-class MyTests.Aspects.Caching
  # In another terminal:
  dotnet-dump collect -p <pid> -o leak.dmp
  ```

  `dotnet-dump` is a global tool: `dotnet tool install -g dotnet-dump`.

Run a small number of tests, so that the dump is small and the leak is still visible.

## Finding the root of a leak

Analyze the dump with `dotnet-dump analyze`. Each `-c` option runs one command, and `-c exit` ends the session, so the analysis works from a script.

1. List the loader allocators. Each leaked compilation has one:

   ```powershell
   dotnet-dump analyze leak.dmp -c "dumpheap -type System.Reflection.LoaderAllocator" -c exit
   ```

2. Find what references each loader allocator:

   ```powershell
   dotnet-dump analyze leak.dmp -c "gcroot <address>" -c exit
   ```

3. Read the chain from its root. The object just before the first type of the compilation is the shared state to fix. Typical roots are:

   | The chain starts with | Cause |
   |---|---|
   | `(static variable: <type>.<field>)` | A static field of a shared assembly. `gcroot` can report the wrong field name: identify the field from the chain of objects. |
   | A `Dictionary<String, Object>` of the framework | `AppDomain.SetData` or `AppContext.SetData`. |
   | A multicast delegate | The handler of an event. |
   | A thread or a timer | A thread or a timer that the compilation started. |

4. Fix the root, and run the check again. One leaked type keeps the whole compilation alive, so the leak disappears only when its last root is gone.

Keep these points in mind when you read the roots:

* Run `gcroot` on the `LoaderAllocator`, not on the assembly load context. An unloaded context is held by a strong handle until its loader allocator is collected, so the only root of the context is that handle.
* Handles that `gcroot` shows with `(10)`, which point from a `RuntimeType` to its `LoaderAllocator`, belong to the loader allocator. They are weak and are not the cause.
* `gcroot` does not report every static event of the framework. When it finds only `(10)` handles, list the objects of your types with `dumpheap -stat -type <your namespace>`. A `<>c` object is the compiler-generated cache of the lambdas of a class: that class converted a lambda to a delegate, possibly to subscribe to an event.

## See Also

**Reference**

<xref:PostSharp.Extensibility.ProjectLocal`1>
<br><xref:PostSharp.Extensibility.PostSharpEnvironment>

**Other Resources**

<xref:aspect-tests>
<br><xref:aspect-test-reference>
<br><xref:debugging-buildtime>
