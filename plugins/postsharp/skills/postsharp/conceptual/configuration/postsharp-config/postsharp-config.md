---
uid: configuration-system
redirect_from:
  - deploymentconfiguration/configuration/configuration-system
title: "Configuring Projects Using postsharp.config"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp can be configured using an XML file named postsharp.config, placed in your project or parent directory. This file can be used for adding a license key, configuring build-time options, adding aspects, and sharing configurations between projects."
---
# Configuring Projects Using postsharp.config

You can configure PostSharp by putting an XML file named *postsharp.config* in your project directory or in any parent directory. This configuration file can also be named `<YourProject>.psproj` or `<YourProject>.pssln`, as described below. 


## When to use a configuration file

Configuration files are typically useful in the following scenarios:

* Adding a license key to your source code:
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
      <License Value="000-AAAAAAAAAAAAAAA"/>
    </Project>
    ```


* Configuring the build-time options of features like logging, for instance:
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
      <Logging xmlns="clr-namespace:PostSharp.Patterns.Diagnostics;assembly:PostSharp.Patterns.Diagnostics">
        <Profiles>
          <LoggingProfile Name="detailed" IncludeSourceLineInfo="True" IncludeExecutionTime="True" IncludeAwaitedTask="True">
            <DefaultOptions>
              <LoggingOptions IncludeParameterType="True" IncludeThisValue="True"/>
            </DefaultOptions>
          </LoggingProfile>
        </Profiles>
      </Logging>
    </Project>
    ```

    See [editing a build-time configuration](xref:logging-customizing#editing-a-build-time-configuration) for details. 

* Adding aspects to your code without affecting your code base, for instance:
    ```xml
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
      <Multicast>
        <LogAttribute xmlns="clr-namespace:PostSharp.Patterns.Diagnostics;assembly:PostSharp.Patterns.Diagnostics"
                      AttributeTargetTypes="PostSharp.Patterns.Diagnostics.Tests.NLog.Person" />
      </Multicast>
    </Project>
    ```

    If you place *postsharp.config* in a parent directory that includes several projects, the aspects will be multicast in all of those projects. You can use this approach to add logging to your entire solution. 
    See <xref:xml-multicasting> for details. 

Other less common use cases are:

* Configuring properties. See the <xref:configuration-schema> for details. 

* Including a plug-in. See the <xref:configuration-schema> for details. 


## Configuration file schema

See <xref:configuration-schema>. 


## Well-known configuration files

PostSharp will automatically load a few well-known configuration files if they are present on the file system, in the following order:

* Any file named `postsharp.config` located in the directory containing the MSBuild project file (*csproj* or *vbproj*, typically), or in any parent directory, up to the root. These files are loaded from the root directory down to the project directory. This is the recommended way to configure new projects. 

* Any file named `<MySolution>.pssln` located in the same directory as the solution file `<MySolution>.sln`. Visual Studio Tools for Metalama and PostSharp also uses this file for convenience. However, we recommend that you do not use this file and move the configuration to a shared *postsharp.config* file instead. The reason is that a project file can belong to several solutions, and it would be misleading to have the compilation depend on which solution was used. 

* Any file named `<MyProject>.psproj` located in the same directory as the project file `<MyProject>.csproj` or `<MyProject>.vbproj`. 

For instance, the files may be loaded in the following order:

* `c:\src\BlueGray\postsharp.config`
* `c:\src\BlueGray\FrontEnd\postsharp.config`
* `c:\src\BlueGray\FrontEnd\BlueGray.FrontEnd.Web\postsharp.config`
* `c:\src\BlueGray\Solutions\BlueGray.pssln` assuming that the current solution file is `c:\src\BlueGray\Solutions\BlueGray.sln`. 

* `c:\src\BlueGray\FrontEnd\BlueGray.FrontEnd.Web\BlueGray.FrontEnd.Web.psproj` assuming that the current project file is `c:\src\BlueGray\FrontEnd\BlueGray.FrontEnd.Web\BlueGray.FrontEnd.Web.csproj`. 


## Sharing configuration between projects

The most typical way to share configuration between projects is to have a *postsharp.config* file in a parent directory of all projects. 

You can also define a `Using` configuration element: 

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.postsharp.org/1.0/configuration">
   <Using File="..\LoggingProfiles.config"/>
</Project>
```

See <xref:configuration-schema> for details.

