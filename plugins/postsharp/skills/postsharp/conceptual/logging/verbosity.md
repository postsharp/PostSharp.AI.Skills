---
uid: log-enabling
redirect_from:
  - logging/log-enabling
title: "Adjusting Logging Verbosity"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on how to adjust logging verbosity in PostSharp, using an XML configuration file or programmatically. It covers configuring logging verbosity, per-request or per-transaction logging, and sampled logging."
---
# Adjusting Logging Verbosity

Sometimes when an issue happens in production, you will want to enable tracing for a specific type or namespace dynamically, without rebuilding the application. To prepare for this scenario, you need to add as much logging as is reasonable at build time, but disable it by default at run time, and then enable it selectively.

You can configure verbosity at run time using an XML configuration file or programmatically, using an API. You can store the XML configuration file online and specify an auto-reload interval.

You can also configure the logging verbosity differently for each request or transaction.

This article shows how to configure verbosity with a configuration file. You can also change it programmatically. For details, see <xref:logging-verbosity-api>. 


## Configuring logging verbosity

Thanks to the `PostSharp.Patterns.Diagnostics.Configuration` package, you can modify your logging verbosity policies dynamically without redeploying or even restarting your application. You only need to create an XML file, store it online with public access (preferably under a secret URL), and call the <xref:PostSharp.Patterns.Diagnostics.LoggingConfigurationManager.ConfigureFromXmlWithAutoReloadAsync(PostSharp.Patterns.Diagnostics.LoggingBackend,System.Uri,System.TimeSpan,System.Threading.CancellationToken)> method. Note that you can also configure verbosity from a local file or without automatic reload using the <xref:PostSharp.Patterns.Diagnostics.LoggingConfigurationManager.ConfigureFromXml*> method. 


To configure logging verbosity with auto reload:

1. Add the `PostSharp.Patterns.Diagnostics.Configuration` package to your top-level project (the one with the `Program` class). 


2. Create a logging verbosity configuration file based on the example below. It sets the default logging level to `warning`, but `debug` for the `Foo` namespace and `trace` for the `Foo.Bar` class. Note that the order of the `source` elements is important: a record overrides the previous ones for anything under its namespace. 

    ```xml
    <logging>
        <verbosity level='warning'>
    		<source name='Foo' level='debug' />
    		<source name='Foo.Bar' level='trace' />
    	</verbosity>
    </logging>
    ```

    For details regarding this file format, see <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.LoggingConfigurationModel>. 


3. Store this file on any file service offering HTTPS access.

    > [!WARNING]
    > The XML file can contain code that executes on the device from which you call this method. A malicious attacker could create a file that causes data breaches or denial of service. Therefore, you must ensure that only authorized personnel have write access to the remote file. Additionally, this file must be publicly readable. Therefore, it should not include any sensitive information (there should be no reason to do so).


4. Finish the configuration of your backend by calling <xref:PostSharp.Patterns.Diagnostics.LoggingConfigurationManager.ConfigureFromXmlWithAutoReloadAsync(PostSharp.Patterns.Diagnostics.LoggingBackend,System.Uri,System.TimeSpan,System.Threading.CancellationToken)>. 


5. Run your application and check the error logs. Mistakes in the policy file will not throw exceptions into your application but report errors to your logs.



## Configuring per-request or per-transaction logging

PostSharp Logging makes it simple to add logging to your application, so you can end up capturing gigabytes of data every minute. Most of this data is never useful, but it causes a performance overhead, and you still need to pay for storage and bandwidth. The ability to trace an application at a high level of detail is very useful, but only if you can select when you want to log. 

For instance, if you are running a web application, it is probably useless to log every single request with the highest level of detail, especially for types of requests that are served 100 times per second. Therefore, it is important to be able to decide, at run time, which requests need to be logged. You may choose to disable logging by default and to enable logging for selected requests only. We call that *per-request* or, more generally, *per-transaction* logging. 

The first thing to do is to define transaction boundaries in your application. The following procedures enable this with ASP.NET Core and the legacy ASP.NET. To define your own transaction type, see <xref:custom-logging-transactions>. 


### Enabling per-request logging for ASP.NET Core

1. Add the `PostSharp.Patterns.Diagnostics.AspNetCore` package to your top-level project (the one with the `Startup` class). 


2. In your `Program.Main` method, call the <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetCore.AspNetCoreLogging.Initialize(PostSharp.Patterns.Diagnostics.Correlation.ICorrelationProtocol,System.Predicate{Microsoft.AspNetCore.Http.HttpRequest},PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata)> method: 

    ```csharp
    AspNetCoreLogging.Initialize();
    ```


3. Define per-transaction logging in your logging verbosity configuration file and set the `type` attribute of the `/logging/transactions/policy` element to `AspNetCoreRequest`, for instance: 

    ```xml
    <logging>
        <verbosity level='warning'/>
        <transactions>
            <policy type='AspNetCoreRequest' 
                    if='t.Request.Path.StartsWith("/invoices")' 
                    name='Policy1'>
                <verbosity>
                    <source level='debug'/>
                </verbosity>
            </policy>
        </transactions>
    </logging>
    ```



### Enabling per-request logging for the legacy IIS-based ASP.NET

1. Add the `PostSharp.Patterns.Diagnostics.AspNetFramework` package to your web application project (the one with the `Global.asax` file). 


2. Add <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetFramework.PostSharpLoggingHttpModule> as an HTTP module in your `Web.config` file. 

    ```xml
    <configuration>
    	<system.webServer>
    		<modules>
    		  <add name="PostSharpLogging" type="PostSharp.Patterns.Diagnostics.Adapters.AspNetFramework.PostSharpLoggingHttpModule"/>
    		</modules>
    	</system.webServer>
    </configuration>
    ```


3. Define per-transaction logging in your logging verbosity configuration file and set the `type` attribute of the `logging/transactions/policy` to `AspNetFrameworkRequest`, for instance: 

    ```xml
    <logging>
        <verbosity level='warning'/>
        <transactions>
            <policy type='AspNetFrameworkRequest' 
                    if='t.Request.Path.StartsWith("/invoices")' 
                    name='Policy1'>
                <verbosity>
                    <source level='debug'/>
                </verbosity>
            </policy>
        </transactions>
    </logging>
    ```


The policy files above (which are identical except for the transaction type) define the default logging level as `warning` and raise the verbosity to `debug` for all incoming HTTP requests whose path starts with `/invoices`.

The most interesting part of the example above is the `if` attribute. This is a C#-like expression that takes a parameter `t` of type <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.OpenTransactionExpressionModel`1> and must return a `bool`. <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.OpenTransactionExpressionModel`1> is a generic type. The type of its <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.OpenTransactionExpressionModel`1.Request> property is determined by the value of the `type` attribute of the `policy` element: 

* For `AspNetCoreRequest`, the type is <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetCore.AspNetCoreRequestExpressionModel>. 

* For `AspNetFrameworkRequest`, the type is <xref:PostSharp.Patterns.Diagnostics.Adapters.AspNetFramework.AspNetFrameworkRequestExpressionModel>. 

* For custom transactions, see <xref:custom-logging-transactions>. 

The `if` attribute is interpreted using [Dynamic Expresso](https://github.com/davideicardi/DynamicExpresso), an expression engine that emulates C#. It accepts the functions defined in the <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.TransactionPolicyExpressionFunctions> class (which you must use without type prefix). The <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.TransactionPolicyExpressionFunctions.Matches(System.String,System.String)> function allows you to match a string against a regular expression. 


## Configuring sampled logging

In the example above, all requests that match the `if` expression are assigned the same logging verbosity. That could mean hundreds of requests per second. Instead of logging all requests that match a given predicate, you can choose to log only a fraction of them. This can be done with the `sample` attribute. In this attribute, you typically call one of these sample functions: <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.TransactionPolicyExpressionFunctions.Random(System.Double)> or <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.TransactionPolicyExpressionFunctions.OnceEveryXSeconds(System.Double,System.String)>. 

The following policy file defines two transaction policies: the first catches a random 10% sample of all ASP.NET Core requests whose path starts with `/invoices`, and the second catches at most one request per minute for all requests whose path starts with `/orders`. Both transactions raise the verbosity to `debug`, but only for those transactions. 

```xml
<logging>
    <verbosity level='warning'/>
    <transactions>
        <policy type='AspNetCoreRequest' 
                if='t.Request.Path.StartsWith("/invoices")' 
                sample='Random(0.1)' 
                name='Policy1'>
            <verbosity>
                <source level='debug'/>
            </verbosity>
        </policy>
        <policy type='AspNetCoreRequest' 
                if='t.Request.Path.StartsWith("/orders")' 
                sample='OnceEveryXSeconds(60, t.Request.Path)' 
                name='Policy2'>
            <verbosity>
                <source level='debug'/>
            </verbosity>
        </policy>
    </transactions>
</logging>
```




