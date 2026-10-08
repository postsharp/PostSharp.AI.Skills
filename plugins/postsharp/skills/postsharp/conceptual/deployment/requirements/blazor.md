---
uid: blazor
redirect_from:
  - deploymentconfiguration/deployment/blazor
title: "Compatibility with Blazor"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp supports Blazor via .NET Standard, but cannot be directly added to a Blazor application project. To prevent code removal, a custom linker configuration is needed."
---
# Compatibility with Blazor

PostSharp supports Blazor as a run-time platform only via .NET Standard. You can use PostSharp in your .NET Standard libraries and then reference these libraries in your Blazor application project. Adding PostSharp directly to a Blazor application project is not supported.

For a complete list of PostSharp packages that support Blazor, see <xref:requirements>.


## Custom linker configuration

Blazor WebAssembly applications use a linker, the IL trimmer, when you publish them. The purpose of the linker is to discard unused code and reduce the size of the application. Linking is based on static analysis, and it cannot correctly detect all the code used by PostSharp. To prevent the linker from removing the required code, you need a custom linker configuration in your project.

To add a custom linker configuration to your Blazor application project, add a new XML file to the project and specify the file as a `TrimmerRootDescriptor` item in the project file:

```xml
<ItemGroup>
  <TrimmerRootDescriptor Include="LinkerConfig.xml" />
</ItemGroup>
```

Then copy the following content to the new XML configuration file:

```xml
<linker>
  <assembly fullname="netstandard">
    <type fullname="*">
    </type>
  </assembly>
</linker>
```

For more detailed information about the linker configuration see the Blazor documentation: [Configure the Trimmer for ASP.NET Core Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/host-and-deploy/configure-trimmer). 


## Debugging

To debug Blazor applications, it is recommended to set the `PostSharpDebuggerExtensionsMode` MSBuild property to `Disabled`, otherwise the debugger may fail to work properly. This should be done for all projects in the solution. The simplest way to achieve this is to [set the property in file Directory.Build.props in the solution root](https://docs.microsoft.com/en-us/visualstudio/msbuild/customize-your-build#directorybuildprops-and-directorybuildtargets).

By default, PostSharp emits debugging information to improve the debugging experience of code enhanced by PostSharp. This includes additional sequence points and other metadata consumed by Visual Studio Debugger extension that is a part of Visual Studio Tools for Metalama and PostSharp.
Other debuggers will not interpret this information and may reject symbols containing it or get into an invalid state. The Blazor debugger is unable to load debugging symbols for assemblies that contain PostSharp's additional debugging information.

## See Also

**Other Resources**

<xref:requirements>
<br>[Configure the Trimmer for ASP.NET Core Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/host-and-deploy/configure-trimmer)
