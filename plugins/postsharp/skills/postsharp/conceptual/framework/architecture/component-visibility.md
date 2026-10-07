---
uid: control-visibility
redirect_from:
  - custompatterns/constraints/tutorials/control-visibility
title: "Controlling Component Visibility Beyond Private and Internal"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp offers the ability to define component access rules beyond the scope of private and internal keywords, allowing developers to restrict access to a component only from other components in the same namespace or specific types. It also supports error escalation and warning suppression."
---
# Controlling Component Visibility Beyond Private and Internal

Sometimes you want to restrict access to a component that you have written. Usually, you control this access with the `private` and `internal` keywords. Any other class in the same assembly can access an `internal` class, but that may not be the level of restriction that the codebase needs. Only the class or struct that contains a `private` class can access it. The first keyword restricts access to the containing class or struct. The second keyword allows access from any other component in the same assembly. What if you need something in between?

PostSharp lets you define component access rules that are between the scope of the `internal` and `private` keywords. You can restrict access to a component to other components in the same namespace. You can also restrict access to a few specific components.

As an example, consider a data access class. As a precaution against developers circumventing the data access structure, you want to limit access to this repository class.


## Restricting access to specific namespaces

> [!NOTE]
> This procedure requires [Visual Studio Tools for Metalama and PostSharp](https://marketplace.visualstudio.com/items?itemName=PostSharpTechnologies.PostSharp) to be installed on your machine. You can however achieve the same results by editing the code and the project manually. 

To limit access to a class to other classes within the validation namespace:

1. Put the caret on the `internal` class that should have restricted access. Select "Add architectural constraint..." from the smart tag options. 

    ![](ComponentInternal_1.png)


2. Select "Prohibit use outside of given types" from the list of options.

    ![](ComponentInternal_2.png)


3. Verify that you will be adding the <xref:PostSharp.Constraints.ComponentInternalAttribute> attribute to the correct piece of code. 

    ![](ComponentInternal_3.png)


4. When the download, installation and configuration of PostSharp have finished, close the wizard and look at the changes that were made to your codebase.

    ![](ComponentInternal_4.png)


5. Notice that the only change in the code is the addition of the [<xref:PostSharp.Constraints.ComponentInternalAttribute>] attribute. 

    ```csharp
    namespace Sharpcrafters.Crm.Console.Repositories 
    { 
        public class InvoiceRepository 
        { 
                [ComponentInternal] 
                internal IEnumerable<Invoice> FetchAllForCustomer(Guid id) 
                { 
                    //dostuff 
                    return null; 
                } 
         } 
    }
    ```


6. Without arguments, the [<xref:PostSharp.Constraints.ComponentInternalAttribute>] attribute allows access only from the namespace of the declaring type and its child namespaces. To grant access to other code, pass arguments to the constructor. There are two options. The first is to pass an array of `typeof(...)` values that represent the types that can access this method. The second is to pass an array of strings that contain the namespaces of the code that can access this method. A namespace also grants access to its child namespaces. For this example, pass a string for the validation namespace. 


7. If you try to access this component from a namespace that has not been granted access, you will see a build-time warning in the Output window.

    ```csharp
    namespace Sharpcrafters.Crm.Console.Services 
    { 
        public class InvoiceServices 
        { 
              public IEnumerable<InvoiceForList> FetchAllInvoicesForCustomer(Guid id) 
              { 
                  var invoiceRepository = new InvoiceRepository(); 
     
                  var allInvoices = invoiceRepository.FetchAllForCustomer(id); 
                  return 
                      allInvoices.Where(x => !x.PaidInFull).Select( 
                          x => new InvoiceForList 
                              { 
                                  PurchaseDate = x.PurchaseDate,  
                                  ShipDate = x.ShipDate,  
                                  TotalAmount = x.Total 
                               }); 
              } 
          } 
    }
    ```

    ![](ComponentInternalWarning.PNG)

    > [!NOTE]
    > If you try to access the component from a namespace that is in a different project, PostSharp must process that project for the validation to occur.



## Restricting access to specific types

Namespace-level restrictions may not be tight enough for your needs. In that case, you can apply this constraint at the type level.

1. To restrict access at the component type level, explicitly define which component types have access. To do this, pass types to the constructor of the <xref:PostSharp.Constraints.ComponentInternalAttribute> attribute. The constructor accepts an array of `Type`, so you can grant access to many different component types. 

    ```csharp
    public class InvoiceRepository 
    { 
        [ComponentInternal(typeof(Sharpcrafters.Crm.Console.Services.InvoiceServices))] 
        internal IEnumerable<Invoice> FetchAllForCustomer(Guid id) 
        { 
            //dostuff 
            return null; 
        } 
    }
    ```


2. If you try to access this component from a type that has not been granted access, you will see a build-time warning in the Output window.

    ```csharp
    public class CustomerServices 
    { 
        public IEnumerable<Customer> FetchAll() 
        { 
            var invoiceRepository = new InvoiceRepository(); 
            var allInvoices = invoiceRepository.FetchAllForCustomer(Guid.NewGuid()); 
            return null; 
        } 
    }
    ```

    ![](ComponentInternalWarning.PNG)



## Controlling component visibility outside of the containing assembly

Because of framework limitations or automated testing requirements, you sometimes need to declare components as public. You probably do not want external applications to access some of those components. For instance, WPF controls need a default constructor for use in the designer, but sometimes you want another constructor to be used at run time, so you want to prevent code from using the default constructor.

PostSharp lets you decorate a publicly declared component so that applications that reference its assembly cannot access it. To do this, apply the <xref:PostSharp.Constraints.InternalAttribute> attribute. 

As with the `internal` keyword, an assembly that the declaring assembly lists in an <xref:System.Runtime.CompilerServices.InternalsVisibleToAttribute> attribute can still use the component without a warning.

1. Mark the `Customer` class so that it can be accessed only from the assembly where it resides. 

    ```csharp
    namespace Sharpcrafters.Crm.Core 
    { 
        public class Customer 
        { 
            public int Id { get; set; } 
            public string Name { get; set; } 
        } 
    }
    ```


2. Place the caret on the publicly declared component that you want to restrict external access to, and expand the smart tag. Select "Add architectural constraint". This procedure requires [Visual Studio Tools for Metalama and PostSharp](https://marketplace.visualstudio.com/items?itemName=PostSharpTechnologies.PostSharp). You can also achieve the same result by editing the code and the project manually. 

    ![](Internal_1.PNG)


3. When prompted to select a constraint, choose to "Prohibit use outside of the project".

    ![](Internal_2.png)


4. On the summary page, review your selections. If the configuration is not what you wanted, click **Previous** and adjust your selections. If the configuration meets your needs, click **Next**. In this example, the [<xref:PostSharp.Constraints.InternalAttribute>] attribute is added to the `Customer` class. 

    ![](Internal_3.png)


5. When the download, installation and configuration of PostSharp have finished, close the wizard and look at the changes that were made to your codebase.

    ![](Internal_4.png)


6. Notice that the only change in the code is the addition of the [<xref:PostSharp.Constraints.InternalAttribute>] attribute. 

    ```csharp
    namespace Sharpcrafters.Crm.Core 
    { 
        [Internal] 
        public class Customer 
        { 
            public int Id { get; set; } 
            public string Name { get; set; } 
        } 
    }
    ```


7. When you use that public component in a different assembly, the build-time warning `AR0105` appears in the Output window. Applying a custom attribute that has this restriction in a different assembly also emits `AR0105`.

    ```csharp
    namespace Sharpcrafters.Crm.Console.Repositories 
    { 
        public class CustomerRepository:ICustomerRepository 
        { 
            public IEnumerable<Customer> FetchAll() 
            { 
                return new List<Customer>{new Customer{Id=1,Name="Joe Johnson"}}; 
            } 
        } 
    }
    ```

    > [!NOTE]
    > PostSharp must process the project that uses the public component for this validation to occur.



## Emitting errors instead of warnings

By default, any code that breaks the access rules defined by the <xref:PostSharp.Constraints.ComponentInternalAttribute> or <xref:PostSharp.Constraints.InternalAttribute> attribute generates a build-time warning. You can escalate this warning to an error. 

1. To change the warning to an error, set the <xref:PostSharp.Constraints.ComponentInternalAttribute.Severity> property. 

    ```csharp
    [ComponentInternal(typeof (InvoiceServices), Severity = SeverityType.Error)] 
    public IEnumerable<Invoice> FetchAllForCustomer(Guid id) 
    { 
        //dostuff 
        return null; 
    }
    ```


2. When you try to access the component without being granted access, the Output window displays an error message.

    ![](ComponentInternalError.PNG)



## Ignoring warnings

In some situations, you may want to suppress the warning that is generated at build time. In those cases, apply the <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute to the locations where you want to allow access to the component. 

> [!NOTE]
> The <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute suppresses only warnings. If you have escalated the warnings to errors, those errors are still generated even if the <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute is present. 

To allow access to the constrained component in a specific method, add the <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute to that method. 

```csharp
public class CustomerServices 
{ 
    [SuppressWarning("AR0102")] 
    public IEnumerable<Customer> FetchAll() 
    { 
        var invoiceRepository = new InvoiceRepository(); 
        var allInvoices = invoiceRepository.FetchAllForCustomer(Guid.NewGuid()); 
        return null; 
    } 
}
```

> [!NOTE]
> `AR0102` is the identifier of the warning emitted by <xref:PostSharp.Constraints.ComponentInternalAttribute>. To ignore warnings emitted by <xref:PostSharp.Constraints.InternalAttribute>, use the identifier `AR0105`. 
>
> <xref:PostSharp.Extensibility.SuppressWarningAttribute> works with any PostSharp warning, not only this one. Any build warning or error, whether from MSBuild, C# or PostSharp, has an identifier. To see identifiers in Visual Studio, open the **View** menu, click **Output**, and select **Show output from: Build**. The warnings are displayed with their identifiers. 

To allow access in an entire class, add the <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute at the class level. The warning is then suppressed for any access to the constrained component within the class. 

```csharp
[SuppressWarning("AR0102")] 
public class CustomerServices 
{ 
    public IEnumerable<Customer> FetchAll() 
    { 
        var invoiceRepository = new InvoiceRepository(); 
        var allInvoices = invoiceRepository.FetchAllForCustomer(Guid.NewGuid()); 
        return null; 
    } 
}
```

## See Also

**Reference**

<xref:PostSharp.Extensibility.SuppressWarningAttribute>
<br><xref:PostSharp.Constraints.ComponentInternalAttribute>
<br><xref:PostSharp.Constraints.InternalAttribute>

**Other Resources**

<xref:custom-constraints>
