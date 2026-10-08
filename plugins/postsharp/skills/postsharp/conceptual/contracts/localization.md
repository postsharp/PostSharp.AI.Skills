---
uid: contract-localization
redirect_from:
  - model/contracts/contract-localization
title: "Localizing Contract Error Messages"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to localize error messages in PostSharp's built-in contracts, using the ContractLocalizedTextProvider class, and also how to localize custom contracts."
---
# Localizing Contract Error Messages

You can customize all texts of exceptions raised by built-in contracts. This allows you to localize error messages into different languages.

Contracts use the <xref:PostSharp.Patterns.Contracts.ContractLocalizedTextProvider> class to obtain the text of an error message. This class follows a simple chain of responsibility pattern where each provider has a reference to the next provider in the chain. When a message is queried, the provider either returns a message or passes control to the next provider in the chain. 

Each message is identified by a string identifier and can refer to four basic arguments and additional arguments specific to a message type. For general information about message arguments see the remarks section of <xref:PostSharp.Patterns.Contracts.LocationContractAttribute>. To find the identifier of a particular message and its additional arguments, see the remarks section of the contract classes in <xref:PostSharp.Patterns.Contracts>. 


## Localizing a built-in error message

The following steps illustrate how to override an error message of a given contract:


### Creating a localized text provider

1. Declare a class that derives from <xref:PostSharp.Patterns.Contracts.ContractLocalizedTextProvider> and implement the chain constructor. 

    ```csharp
    public class CzechContractLocalizedTextProvider : ContractLocalizedTextProvider
    {
        public CzechContractLocalizedTextProvider(ContractLocalizedTextProvider next)
          : base(next)
        {
        }
        
    }
    ```


2. Implement the <xref:PostSharp.Patterns.Utilities.LocalizedTextProvider.GetMessage(System.String)> method. In the next code snippet, we show how to build a simple and efficient dictionary-based implementation. 

    ```csharp
    public class CzechContractLocalizedTextProvider : ContractLocalizedTextProvider
    {
        private readonly Dictionary<string, string> messages = new Dictionary<string, string>
        {
            {RegularExpressionErrorMessage, "Hodnota {2} neodpovídá regulárnímu výrazu '{4}'."},
        };

        public CzechContractLocalizedTextProvider(ContractLocalizedTextProvider next)
          : base(next)
        {
        }
        
        public override string GetMessage( string messageId )
        {
            if ( string.IsNullOrEmpty( messageId ))
                throw new ArgumentNullException(nameof(messageId));
    
            string message;
            if ( this.messages.TryGetValue( messageId, out message ) )
            {
                return message;
            }
            else
            {
                // Fall back to the default provider.
                return base.GetMessage( messageId );
            }
        }
    }
    ```

    > [!NOTE]
    > If you need to support several languages, you can make your implementation of the <xref:PostSharp.Patterns.Utilities.LocalizedTextProvider.GetMessage(System.String)> method depend on the value of the <xref:System.Globalization.CultureInfo.CurrentCulture> property. You can optionally store your error messages in a managed resource and use the <xref:System.Resources.ResourceManager> class to access it and manage localization issues. The design of PostSharp Code Contracts is agnostic to these decisions. 


3. At the beginning of the application, create a new instance of the provider and set the current provider as its successor.

    ```csharp
    public static void Main()
    {
        ContractServices.LocalizedTextProvider = new CzechContractLocalizedTextProvider(ContractServices.LocalizedTextProvider);
    
        // ...
    }
    ```



## Localizing custom contracts

Once you have configured a text provider, you can use it to localize error messages of custom contracts. In the following procedure, we will localize the error message of the example contract described in <xref:custom-contracts>. 


### Adding the message of a custom contract

1. Edit the code contract class (`NonZeroAttribute` in our case) to call `ContractServices.ExceptionFactory.CreateException()` in the validation method(s), as shown in <xref:contract-custom-exceptions>. Use a unique `messageId` (for example, "NonZeroErrorMessage") to identify the message text template. 


2. Edit your text provider class (`CzechContractLocalizedTextProvider` in our case) and include the message for your custom contract: 

    ```csharp
    private readonly Dictionary<string, string> messages = new Dictionary<string, string>
      {
         {RegularExpressionErrorMessage, "Hodnota {2} neodpovídá regulárnímu výrazu '{4}'."},
         {"NonZeroErrorMessage", "Hodnota {2} nesmí být 0."}
      };
    ```



