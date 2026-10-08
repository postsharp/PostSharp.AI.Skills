---
uid: postsharp-unattended
redirect_from:
  - deploymentconfiguration/deployment/postsharp-unattended
title: "Installing PostSharp Unattended"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "This document provides a detailed guide on how to install PostSharp, a user interface for Visual Studio, unattended using a script. It also includes a PowerShell 2.0 script for automation."
---
# Installing PostSharp Unattended

PostSharp is composed of a user interface (Visual Studio Tools for Metalama and PostSharp) and build components (NuGet packages). NuGet packages are usually checked into source control or retrieved from a package repository at build time (see <xref:nuget-restore>), so their deployment does not require additional automation. The user interface is typically installed by each user. It does not require administrative privileges. 

You can install PostSharp automatically for a large number of users using a script.

## Installing Visual Studio Tools for Metalama and PostSharp on a machine

To install Visual Studio Tools for Metalama and PostSharp on a machine:

1. Execute the command `VsixInstaller.exe /q PostSharpMetalama-VERSION.vsix`. You can download the file from [Visual Studio Gallery](https://marketplace.visualstudio.com/items?itemName=PostSharpTechnologies.PostSharp). Treat exit codes other than `0` or `1001` as errors. 

1. Install the license key or the license server URL in the registry key `HKEY_CURRENT_USER\Software\SharpCrafters\PostSharp 3`, registry value `LicenseKey` (type `REG_SZ`). 

This procedure can be automated by the following PowerShell 2.0 script:

```powershell
# TODO: Set the right value for the following variables

# Replace with the proper version number and add the full path.
$postsharpFile = "PostSharpMetalama.2024.1.11.vsix"

# Replace by your license key or license server URL.
$license = "XXXX-XXXXXXXXXXXXXXXXXXXXXXXXX"

# Replace the path to the Visual Studio installation with the actual path on your system.
$vsixInstaller = "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\Common7\IDE\VsixInstaller.exe"

# Install Visual Studio Tools for Metalama and PostSharp
Write-Host "Installing Visual Studio Tools for Metalama and PostSharp"
$process = Start-Process -FilePath $vsixInstaller -ArgumentList @("/q", $postsharpFile) -Wait -PassThru
if ( $process.ExitCode -ne 0 -and $process.ExitCode -ne 1001)
{
    Write-Host "Error: VsixInstaller exited with code" $process.ExitCode -ForegroundColor Red
}

# Install the license key
Write-Host "Installing the license key"
$regPath = "HKCU:\Software\SharpCrafters\PostSharp 3"

if ( -not ( Test-Path $regPath ) )
{
    New-Item -Path $regPath | Out-Null
}

Set-ItemProperty -Path $regPath -Name "LicenseKey" -Value $license

Write-Host "Done"
```