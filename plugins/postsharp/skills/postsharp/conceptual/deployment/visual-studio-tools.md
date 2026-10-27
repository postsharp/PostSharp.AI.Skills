---
uid: installation
redirect_from:
  - deploymentconfiguration/deployment/installation
title: "Installing Visual Studio Tools for Metalama and PostSharp"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions on how to install Visual Studio Tools for Metalama and PostSharp."
---
# Installing Visual Studio Tools for Metalama and PostSharp

Visual Studio Tools for Metalama and PostSharp are PostSharp's user interface. Installing them on a developer's computer does not affect the projects until the PostSharp NuGet package has been added to a project. See <xref:install-compiler> for details.


To install Visual Studio Tools for Metalama and PostSharp:

1. Download the VSIX package from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=PostSharpTechnologies.PostSharp).

2. Run the `.vsix` file.

3. Complete the installation wizard.

4. Open any solution in Visual Studio and complete the initial setup wizard.

## Outside Visual Studio

Visual Studio Tools for Metalama and PostSharp show which aspects enhance a declaration in CodeLens and tooltips. To get the same information outside Visual Studio, for instance on Linux or macOS, or for an AI coding agent, use the `postsharp introspect` commands, which read the files of the last build. See <xref:introspect>.

## See Also

**Other Resources**

<xref:install-compiler>
<br><xref:uninstalling>
<br><xref:introspect>
