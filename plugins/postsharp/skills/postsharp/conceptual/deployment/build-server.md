---
uid: build-server
redirect_from:
  - deploymentconfiguration/deployment/build-server
title: "Using PostSharp on a Build Server"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp needs no installation or configuration on a build server. This article lists the ways to provide a license key to the build."
---
# Using PostSharp on a Build Server

PostSharp is designed for frictionless use on build servers. PostSharp build-time components are deployed as NuGet packages and are integrated with MSBuild. No component needs to be installed or configured on the build server, and no extra build step is necessary. If you choose not to check in NuGet packages in your source control, read <xref:nuget-restore>. 


## Installing a License on the Build Server

There are several ways to install a license on the build server:

* Do not install it. A build server does not need a license key unless your projects use PostSharp Logging. This works only when PostSharp detects that the build is unattended; see <xref:unexpected-license-usage> for the conditions.

* Add your license key to the *postsharp.config* file and add this file to the source repository as described in <xref:deploying-keys>. 

* Set an environment variable named `PostSharpLicense` to a semicolon-separated list of your license keys. 

> [!NOTE]
> An unattended build never uses a license server, so that build servers do not take seats from developers. PostSharp ignores license server URLs in unattended builds. To build projects that use PostSharp Logging on a build server, use a license key, not a license server.

We do not recommend installing the license key on a build server using the user interface.

## See Also

**Other Resources**

<xref:installation>
<br><xref:deploying-keys>
