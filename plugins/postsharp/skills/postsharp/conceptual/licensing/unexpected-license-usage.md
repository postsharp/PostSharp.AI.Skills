---
uid: unexpected-license-usage
redirect_from:
  - deploymentconfiguration/licensing/unexpected-license-usage
title: "Investigating Unexpected License Usage"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "When more people build with PostSharp than your subscription covers, the two usual causes are a license key that has been copied outside the team that bought it, and developers who only build code they did not write. This article explains how to check both, and describes the perpetual rights that can cover an older build for more users."
---
# Investigating Unexpected License Usage

PostSharp reports the use of license keys back to us, as described in <xref:license-audit>. When the number of people building with your license keys is consistently higher than your subscription covers, we write to the account to tell you.

If you have received such a message and the number surprises you, this article covers the two usual causes and how to check them. It also covers a third cause, which is less frequent.

The message we send links to a page that shows the same daily figures, so you can see how the count was reached.


## How the count works

Your subscription covers a number of users. We count each person once per calendar day, regardless of the number of machines, projects or license keys that this person uses. Two people who build on different days therefore occupy one user, and one person building on four machines is still one user.

A license key identifies your account. It does not carry the number of users that you are entitled to. That number comes from the subscriptions recorded on your account, and it changes when you add users to a subscription. Your developers keep building with the key they already have.

We aggregate the reported usage on our servers, per product and per product version, over all the license keys that belong to your account. The key that a given developer used therefore does not change the total.

Builds running unattended on a build server are not counted, whether or not a license key is installed there. PostSharp detects an unattended build and uses an internal license for it, before it reads any key of yours. That internal license is not audited, and it covers every feature. See <xref:build-server>.

PostSharp treats a build as unattended when it does not run interactively, when it runs in a container, or when its parent process is a known build agent. The known build agents include TeamCity, Jenkins, Azure Pipelines, GitHub Actions, GitLab CI, Bamboo, GoCD, Buildkite, CircleCI, Semaphore and Travis. A build machine that matches none of these conditions, such as a scheduled build on somebody's desktop computer, is counted like any other machine.

The figure we quote excludes your busiest days of the year, so that it represents an ordinary working day.


## Cause 1: a license key used outside the team that bought it

This is the most common cause.

A license key is a text string, and it works on every machine where it is installed. Everyone who builds with your key is counted against your subscription, because we cannot distinguish these builds from the builds of your own developers.

What to check:

* Which repositories contain the key. Search your source control for it, including branches and history, and including repositories that belong to other parts of your organization.

* Which container images and build templates embed it.

* Whether the key was ever sent to a contractor, an outsourcing partner, a sister company or a customer. A key shared for a short project stays valid until the subscription ends.

* Whether a repository that contains the key has been forked, made public, or handed to another team.

See <xref:deploying-keys> for the supported ways to install a key. Some of them keep the key out of source control.

If you find that the key has been copied further than intended, contact us. A key can be replaced.


## Cause 2: developers who only build the code are not excluded

A license is required only to create or modify code that uses PostSharp. Developers who only build code written by somebody else do not need one.

PostSharp does not distinguish these developers by default. Before PostSharp 2027.0, two conditions must both be met before it stops counting them. From PostSharp 2027.0, the first condition is enough.

First, the source control check must be enabled, by setting the `VcsCheckEnabled` property in your *postsharp.config* file. The check works with Git, and with Team Foundation Version Control before PostSharp 2027.0. It requires debug symbols, because PostSharp reads the list of source files from them. See <xref:licensing-shared-source-code> for the procedure and its limitations.

Second, before PostSharp 2027.0, the license key must not reach the developers who only build the code. These versions satisfy a license requirement from an installed key before they read the source control status. If these developers have a key, that key is used and their builds are counted. In practice, this means that the key must not be stored in the repository they clone. From PostSharp 2027.0, PostSharp performs the source control check before it reads any license key, and it reads no key for an unmodified project.

This affects future builds only. Usage that has already been recorded does not change.


## Cause 3: builds covered by an earlier subscription

A per-developer subscription is perpetual. It covers every build of the product released before the end of the subscription, for the number of users of that subscription. This right does not expire when the subscription ends.

If one of your earlier subscriptions covered more users than your current one, the builds released before the end of that earlier subscription are still covered for the higher number. Usage of such a build can therefore be higher than the number of users you are entitled to today, without exceeding your rights.

Successive subscriptions are not added together. For a given build, the number of covered users is the highest of the subscriptions that cover it, not their sum.

This cause is rare, because it requires both an earlier subscription for more users and the continued use of a build from that period. The figures we quote do not take it into account. Tell us if it applies to your organization, and we will analyze the builds concerned.


## Preventing the situation with a license server

The license server removes the license key from the developers' machines. Each build leases a license from a server that you host. The server grants no more leases than your subscription covers, after a configurable grace period. It also shows which users and which machines currently hold a lease, so an unexpected number can be traced to named users.

The license server is available to enterprise customers whose support level makes them eligible. See our [service level agreement](https://postsharp.net/support/policies/sla) for the eligibility conditions, and <xref:license-server-admin> for the installation and administration procedure.


## If none of these explains it

Reply to the message we sent, or contact us at [hello@postsharp.net](mailto:hello@postsharp.net), and we will go through the figures with you. We cannot see the following situations from our side:

* Developers who should be covered by a subscription held by another company in your group, but who use a key issued to this account.

* A subscription held under a different company name that we have not connected to this account.

* A build machine that PostSharp does not recognize as unattended, and therefore counts as a person.

If the team has grown, you can add users to your subscription from your account page.


## See Also

**Other Resources**

<xref:license-audit>
<br><xref:licensing-shared-source-code>
<br><xref:deploying-keys>
<br><xref:license-server-admin>
<br><xref:build-server>
