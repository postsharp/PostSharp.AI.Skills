---
uid: logging-troubleshooting
title: "Troubleshooting PostSharp Logging"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides troubleshooting steps for issues related to PostSharp Logging, focusing on checking code enhancement at build time and verifying run-time setup."
---

# Troubleshooting PostSharp Logging
If you think you have followed all steps but records do not flow to your log, consider the following steps.


## Step 1. Does PostSharp Logging enhance your code?

The first step is to check whether PostSharp Logging works at build time.

1. Compile your project.
2. Open the project binary using a decompiler such as ILSpy.
3. Go to a method that you are sure must be logged.
4. Check the decompiled code. Does it include PostSharp-generated logging code, or is it essentially your source code?

If the decompiled code does not include PostSharp-generated logging code, check the following:

* Is the `PostSharp.Patterns.Diagnostics` package correctly installed in the project? The `PostSharp.Patterns.Diagnostics.Redist` package is not enough.
* Are you sure that the `SkipPostSharp` MSBuild property is not set for this project?
* Are you sure that your multicast attributes are correct? To be sure, explicitly use the `[Log]` attribute on a select method and test again.

## Step 2. Check the run-time setup

If the previous step was successful, that is, if you can see PostSharp-generated logging code, but you still cannot see any record in the log, the problem may be in the run-time configuration.

* Verify that you have followed the instructions to set up the specific logging backend. See <xref:backends> for details.
* Verify that the verbosity level in your logging backend is set to a low enough value. Try to enable the most detailed verbosity level and only reduce it after you see that logging is working.
* Verify that the verbosity level in PostSharp Logging is set to a high enough value. For details, see <xref:log-enabling>.
