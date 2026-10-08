---
uid: assembly-binding-resolution
redirect_from:
  - deploymentconfiguration/configuration/assembly-binding-resolution
title: "Resolution of Assembly Binding Redirections"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp follows assembly binding redirections at build time and stores the assembly binding redirection in a file named PostSharpHost.config. Users can override the default assembly redirections if required."
---
# Resolution of Assembly Binding Redirections

PostSharp executes your project assemblies at build time. This is why PostSharp must follow assembly binding redirections at build time. Although the default assembly redirection mechanism works fine in most cases, there may be situations where you will need to override it.

> [!NOTE]
> To learn why PostSharp executes your project assemblies at build time, see <xref:aspect-lifetime>. 


## Default assembly binding redirections

To follow the same assembly binding redirections as your application does at run time, PostSharp analyzes your projects and configuration files (typically *app.config* or *web.config*) and generates assembly binding redirection configuration. PostSharp stores the assembly binding redirection in a file named *postsharp.app.config* in the *PostSharp* subdirectory of the intermediate output directory (for instance *obj\Debug\PostSharp*). You can review the *postsharp.app.config* file to see what configuration PostSharp uses to resolve assemblies. For an empty ASP.NET MVC 5 application, the *postsharp.app.config* file may look like this: 

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="Microsoft.Owin.Security" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="0.0.0.0-3.0.1.0" newVersion="3.0.1.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="Microsoft.Owin.Security.OAuth" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="0.0.0.0-3.0.1.0" newVersion="3.0.1.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="Microsoft.Owin.Security.Cookies" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="0.0.0.0-3.0.1.0" newVersion="3.0.1.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="Microsoft.Owin" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="0.0.0.0-3.0.1.0" newVersion="3.0.1.0" />
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>
```

> [!NOTE]
> PostSharp relies on the [GenerateBindingRedirects](https://msdn.microsoft.com/en-us/library/microsoft.build.tasks.generatebindingredirects.aspx) MSBuild task. 


## Overriding default assembly redirections

If the default mechanism does not work, you can disable it by setting the MSBuild property `PostSharpDisableDefaultBindingRedirects` to `True` in your project file: 

```xml
<PropertyGroup>
  <PostSharpDisableDefaultBindingRedirects>True</PostSharpDisableDefaultBindingRedirects>
</PropertyGroup>
```

With this configuration, PostSharp does not analyze assembly binding redirections.

> [!WARNING]
> Do not set `PostSharpDisableDefaultBindingRedirects` to `True` unless it is necessary. It may produce results that are difficult to predict. 

> [!NOTE]
> The default algorithm applies only to projects that target .NET Framework. It is disabled for projects that target .NET Core, .NET 5 and later, or .NET Standard.

If you disable default binding redirections, you may want to specify a file with your own assembly binding redirection configuration:

```xml
<PropertyGroup>
  <PostSharpDisableDefaultBindingRedirects>True</PostSharpDisableDefaultBindingRedirects>
  <PostSharpHostConfigurationFile>web.config</PostSharpHostConfigurationFile>
</PropertyGroup>
```

With this configuration, PostSharp uses explicit assembly binding redirection configuration from the `web.config` file. 


