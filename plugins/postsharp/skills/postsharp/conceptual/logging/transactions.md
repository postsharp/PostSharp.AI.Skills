---
uid: custom-logging-transactions
redirect_from:
  - logging/custom-logging-transactions
title: "Defining Your Own Logging Transactions"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a step-by-step guide to defining custom logging transactions in applications not based on ASP.NET using the codeless configuration system of PostSharp."
---
# Defining Your Own Logging Transactions

You can use the codeless configuration system even if your application is not based on ASP.NET. For instance, you can define transactions in a service processing requests from a queue, or files from an input directory.

This article explains how to do it.


To define custom transactions and use them in the policy XML file:

1. Choose a name for your transaction type, for instance `MorningBatch`. 


2. Create the *expression model* class, for instance `MorningBatchExpressionModel`. That will become the type of the <xref:PostSharp.Patterns.Diagnostics.Transactions.Model.OpenTransactionExpressionModel`1.Request> property that is visible to the XML policy file. 

    [!code-csharp[](../../code/samples/Logging/LoggingCustomTransactions/LoggingCustomTransactions.cs#expression-model)]


3. Derive a class from <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata> and use your expression model type as the generic argument. Expose a singleton instance of this class. 

    [!code-csharp[](../../code/samples/Logging/LoggingCustomTransactions/LoggingCustomTransactions.cs#metadata)]


4. In your source code, find the place where the transaction starts and ends. Create an <xref:PostSharp.Patterns.Diagnostics.OpenActivityOptions> instance and pass a <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventData> created from your singleton <xref:PostSharp.Patterns.Diagnostics.Custom.LogEventMetadata>. 


5. Call the <xref:PostSharp.Patterns.Diagnostics.LogSource.ApplyTransactionRequirements(PostSharp.Patterns.Diagnostics.OpenActivityOptions@)> method. This will apply the logging policies to the <xref:PostSharp.Patterns.Diagnostics.OpenActivityOptions>. You can skip this method call and force a transaction to be opened by directly setting the <xref:PostSharp.Patterns.Diagnostics.OpenActivityOptions.TransactionRequirement> property. 


6. Call the <xref:PostSharp.Patterns.Diagnostics.Custom.LogLevelSource.OpenActivity``1(``0@,PostSharp.Patterns.Diagnostics.OpenActivityOptions@)> method at the beginning of your transaction. 


7. Call <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetSuccess(PostSharp.Patterns.Diagnostics.CloseActivityOptions@)> or <xref:PostSharp.Patterns.Diagnostics.LogActivity`1.SetException(System.Exception,PostSharp.Patterns.Diagnostics.CloseActivityOptions@)> at the end. 

    Your transaction code will now look like this:

    [!code-csharp[](../../code/samples/Logging/LoggingCustomTransactions/LoggingCustomTransactions.cs#transaction)]


8. You can now create a configuration file that references your expression model. For instance, the following file will enable logging for batch files larger than a thousand bytes:

    [!code-xml[](../../code/samples/Logging/LoggingCustomTransactions/logging.xml)]

    With a file of 100 bytes and a file of 2000 bytes, only the processing of the large file is logged:

    [!code-text[](../../code/samples/Logging/LoggingCustomTransactions/LoggingCustomTransactions.txt)]



