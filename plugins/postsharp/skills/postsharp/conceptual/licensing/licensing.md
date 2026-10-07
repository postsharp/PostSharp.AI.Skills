---
uid: licensing
redirect_from:
  - deploymentconfiguration/licensing
  - license-types
  - per-user-licenses
title: "Licensing"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides technical information on licensing of PostSharp, including license types, installation of license keys, auditing, shared source code, user counting, server administration, and troubleshooting."
---
# Licensing

This chapter addresses some technical questions related to the licensing of PostSharp. For business questions, see our [website](https://www.postsharp.net/purchase/faq). 


## License types

PostSharp is offered under the following types of license:

* Per-developer licenses limit the number of unique daily users. The number of repositories and the number of enhanced lines of code are unlimited. A per-developer license is a floating license. It is not assigned to named users, and every licensed user has access to all features of the licensed product.

* PostSharp Essentials is the free edition. Its free features can be used on an unlimited number of enhanced lines of code. Its premium features are limited to 1,000 enhanced lines of code. See <xref:express-limitations>.

A per-developer subscription is perpetual. It lets you use any version of PostSharp released before the expiration of the subscription, without time limit.

Per-developer licenses do not require the registration of a device or the activation of a license key. The whole team uses a common license key. See <xref:deploying-keys> for the ways to install it, and <xref:license-audit> for the way its use is enforced.


## Topics in this chapter

This chapter contains the following topics:

| Chapter | Description |
|---------|-------------|
| <xref:deploying-keys> | This topic shows how each developer can install the license key using the UI and how this can be done centrally by adding the license key to source control. |
| <xref:licensing-shared-source-code> | This topic explains that you do not need to purchase a license if you need to build code that you did not write yourself but that uses PostSharp. |
| <xref:unexpected-license-usage> | This topic explains how the number of users is counted, and how to investigate a number that is higher than expected. |
| <xref:license-audit> | This topic explains that PostSharp does not validate the license key in a blocking way, but audits the use of license keys. |
| <xref:license-server-admin> | This topic describes how system administrators can install and maintain the license server. |
| <xref:express-limitations> | This topic describes PostSharp Essentials, the free edition of PostSharp, and its limitations. |
| <xref:license-troubleshooting> | This topic addresses the most common license issues you may encounter. |

