---
uid: license-audit
redirect_from:
  - deploymentconfiguration/licensing/license-audit
title: "License Audit"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp uses an asynchronous, fail-safe license audit instead of license activation, auditing license keys' use on each client machine and periodically reporting to their license servers. It does not require a permanent network connection and will not fail if the license server is unavailable. "
---
# License Audit

Although most software packages are protected with a license activation mechanism, we think that the practice is not adequate for software development tools:

* The source code is sometimes compiled several years after it has been written, and there is no guarantee that the license activation server will still be functional.

* Development teams want their tools to be included in the source control repository together with the source code, and want the license key to be deployed the same way.

Instead of license activation, PostSharp relies on asynchronous, fail-safe license audit. PostSharp audits the use of license keys on each client machine and periodically reports it to our license servers. The mechanism does not require a permanent network connection, and PostSharp will not fail if the license server is not available.

The licensing client will contact our licensing servers in the following cases:

* When a license is registered on a computer with the user interface.

* Once per day, for every user and every device using PostSharp.

No personally identifiable information is transmitted during this process except the license key. In case we suspect a serious violation of the License Agreement, we reserve the right to contact the legitimate owner of this license.

> [!TIP]
> If license audit is not acceptable in your company, contact us with a request to disable license audit. Our sales teams will evaluate your request and answer with a license key containing an audit waiver. Global licenses and site licenses are not subject to license audit by default. The use of the license server does not implicitly disable license audit. For more information, see <xref:license-server-admin>. 

## See Also

**Other Resources**

<xref:license-server-admin>
