---
uid: licensing-shared-source-code
redirect_from:
  - deploymentconfiguration/licensing/licensing-shared-source-code
title: "Sharing Source Code With Unlicensed Teams"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp requires a license to create or modify code, but not to build it. When the VcsCheckEnabled property is set, PostSharp reads the status of your source control repository and does not require a license for unmodified source code."
---
# Sharing Source Code With Unlicensed Teams

A license is required to create or modify code that uses PostSharp. A license is not required to build code that uses PostSharp but that you did not write yourself.

PostSharp distinguishes the two situations by reading the status of your source control repository. If the working copy contains modifications relative to the base commit, PostSharp considers that you are creating or modifying the code, and requires a valid license.

The source control check is disabled by default, because it executes an external process at every build. As long as it is disabled, PostSharp requires a valid license for every build. You can enable the check in the PostSharp configuration file of your project or solution. See <xref:configuration-system>.


## Enabling the source control check

To enable source code sharing with unlicensed teams:

1. Open the file *postsharp.config* that is located in the root directory of your solution or project. If the file does not exist, create a new *postsharp.config* file in that location with the following content: 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
    </Project>
    ```


2. Add a `Property` element under the `Project` element, set the `Name` attribute to `VcsCheckEnabled` and the `Value` attribute to `True`. 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project xmlns="http://schemas.postsharp.org/1.0/configuration">
        <Property Name="VcsCheckEnabled" Value="True" />
    </Project>
    ```


## How PostSharp determines that a project is unmodified

When the check is enabled, PostSharp performs it once per compilation, before it reads any license key. If the project is unmodified, PostSharp reads no license key, takes no lease from a license server, and reports no license usage.

The check proceeds in the following steps:

1. PostSharp reads the list of source files of the compiled assembly from its debug symbols.

2. For each source file, PostSharp looks for a Git repository in the parent directories. A directory that contains a `.git` subdirectory or a `.git` file identifies a repository. Source files that belong to no repository are ignored, but at least one repository must be found.

3. For each repository, PostSharp executes `git status` once, in the root directory of the repository. PostSharp caches the result between builds while the repository does not change.

4. PostSharp then looks for the source files of the current compilation in the output of the command. The project is considered modified when the content of one of these files is modified, whether the change is staged or not. Untracked, added, deleted and renamed files do not count. Changes to files that are not compiled into the current project do not make the project modified.

If the project is unmodified, the compilation requires no license.


## Requirements and limitations

* Git is the only supported source control system. If you use another system, PostSharp always requires a license at build time. Earlier versions of PostSharp also supported Team Foundation Version Control.

* The `git` command must be present in the `PATH` environment variable of the build process. Alternatively, set the `POSTSHARP_GIT_PATH` environment variable to the path of the `git` executable. If PostSharp cannot execute the command, the project is considered modified.

* The debug symbols of the compiled assembly must be available, because PostSharp reads the list of source files from them. If the project is compiled without debug symbols, PostSharp cannot determine which source files to check, and the project is considered modified.

On a build server, you do not need this feature. PostSharp detects unattended builds and grants a different internal license in this situation. See <xref:build-server>.


## Diagnosing the source control check

PostSharp writes the result of the source control check to the licensing trace. See [Capturing licensing diagnostic logs](xref:license-troubleshooting#capturing-licensing-diagnostic-logs) for the procedure.

The trace reports whether the project was found modified. It also reports the situations in which the check could not be performed, for instance when the debug symbols name no source file, when no repository was found, or when the `git` command could not be executed.


## See Also

**Other Resources**

<xref:license-audit>
<br><xref:unexpected-license-usage>
<br><xref:build-server>
<br><xref:configuration-system>
