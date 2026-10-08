---
uid: what-is-postsharp
redirect_from:
  - introduction/overview/what-is-postsharp
title: "Quick Examples"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp is a .NET tool that provides ready-made implementations of common patterns and threading models. It also allows users to create and validate custom patterns using aspect-oriented programming (AOP)."
---
# Quick Examples


## Standard patterns

PostSharp provides implementations of some of the patterns that are most commonly found in .NET code bases:

* INotifyPropertyChanged: see <xref:inotifypropertychanged>. 

* Parent/child relationships: see <xref:aggregatable>. 

* Code contracts: see <xref:contracts>. 

* Logging: see <xref:logging>. 


### Example

The following code snippet illustrates an object model where <xref:System.ComponentModel.INotifyPropertyChanged>, code contracts, and aggregation are all implemented using PostSharp ready-made attributes. 

```csharp
[NotifyPropertyChanged]
public class CustomerViewModel
{
   [Required]
   public Customer Customer { get; set; }
   
   public string FullName { get { return this.Customer.FirstName + " " + this.Customer.LastName; } }
}

[NotifyPropertyChanged]
public class Customer
{
   public string FirstName { get; set; }
   public string LastName { get; set; }
   
   [Child]
   public AdvisableCollection<Address> Addresses { get; set; }
   
   [Url]
   public string HomePage { get; set; }
   
   [Log]
   public void Save(DbConnection connection)
   {
      // ...
   }
}

[NotifyPropertyChanged]
public class Address
{
   [Parent]
   public Customer Parent { get; private set; }
   
   public string Line1 { get; set; }
}
```


## Thread safety patterns

Multithreading is a great demonstration of the limitations of conventional object-oriented programming. Thread synchronization is traditionally addressed at an absurdly low level of abstraction, resulting in excessive complexity and defects.

Yet, several design patterns exist to reduce the complexity of multithreading. New programming languages have been designed around these patterns: for instance Erlang over the Actor pattern and functional programming over the Immutable pattern.

PostSharp gives you the benefits of threading design patterns without leaving C# or VB.

PostSharp supports the following threading models and features:

* Immutable: see <xref:immutable>. 

* Freezable: see <xref:freezable>. 

* Actor: see <xref:actor>. 

* Reader/Writer Synchronized: see <xref:reader-writer-synchronized>. 

* Synchronized: see <xref:synchronized>. 

* Thread Unsafe: see <xref:thread-unsafe>. 

* Thread Affine: see <xref:thread-affine>. 

* Thread Dispatching: see <xref:background-dispatching> and <xref:ui-dispatching>. 


### Example

The following code snippet shows how a data transfer object can be made freezable, recursively but easily:

```csharp
[Freezable]
public class Customer
{
   public string Name { get; set; }
   
   [Child]
   public AdvisableCollection<Address> Addresses { get; set; }
   
}

[Freezable]
public class Address
{
   [Parent]
   public Customer Parent { get; private set; }
   
   public string Line1 { get; set; }
}

public class Program
{
   public static void Main()
   {
      Customer customer = ReadCustomer( "http://customers.org/11234" );
      
      // Prevent changes.
      ((IFreezable)customer).Freeze();
      
      // The following line will cause an ObjectReadOnlyException.
      customer.Addresses[0].Line1 = "Here";
   }
}
```


## Implementation of custom patterns

The attributes that implement the standard and thread safety patterns are called *aspects*. This term comes from the paradigm of *aspect-oriented programming* (AOP). An *aspect* is a class that encapsulates behaviors that are injected into another class, method, field, property or event. The process of injecting an aspect into another piece of code is called *weaving*. PostSharp weaves aspects at build time; it is also called a *build-time aspect weaver*. 

PostSharp Aspect Framework is a pragmatic implementation of AOP concepts. All ready-made implementations of patterns are built using PostSharp Aspect Framework. You can use the same technology to automate the implementation of your own patterns.

To learn more about developing your own aspects, see <xref:custom-aspects>. 


### Example

The following code snippet shows a simple `[PrintException]` aspect that writes an exception message to the console before rethrowing it: 

```csharp
[PSerializable]
class PrintExceptionAttribute : OnExceptionAspect
{
    public override void OnException(MethodExecutionArgs args)
    {
        Console.WriteLine( args.Exception.Message );
    }
}
```

In the next snippet, the `[PrintException]` aspect is applied to a method: 

```csharp
class Customer
{
    public string FirstName { get; set; }
    public string LastName { get; set; }

    [PrintException]
    public void Store(string path)
    {
        File.WriteAllText( path, string.Format( "{0} {1}", this.FirstName, this.LastName ) );
    }
}
```


## Validation of custom patterns

Not all patterns can be fully implemented by the compiler. Many patterns involve a lot of handwritten code. However, they are still patterns because we want to follow the same conventions and approach when solving the same problem. In this case, we have to validate the code against implementation guidelines of the pattern. This is typically achieved during code reviews, but like any algorithmic work, it can be partially automated using the right tool. This is the job of the *PostSharp Architecture Framework*. 

PostSharp Architecture Framework also contains pre-built architectural constraints that help to solve common design problems. For instance, the <xref:PostSharp.Constraints.InternalImplementAttribute> constraint prevents an interface from being implemented in an external assembly. 

See <xref:constraints> for more details about architecture validation. 


### Example

Consider a form-processing application. There may be hundreds of forms, and each form can have dozens of business rules. To reduce complexity, the team decides that all business rules will respect the same pattern. The team decides that each class representing a business rule must contain a public nested class named `Factory`, and that this class must have an `[Export(typeof(IBusinessRuleFactory))]` custom attribute and a default public constructor. The team wants all developers to follow the convention. Therefore, the team decides to create an architectural constraint that will validate the code against the project-specific *Business Rule Factory* pattern. 

```csharp
[MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
public class BusinessRulePatternValidation : ScalarConstraint 
{ 
    public override void ValidateCode(object target) 
    { 
        var targetType = (Type)target; 

        if (targetType.GetNestedType( "Factory" ) == null) 
        { 
            Message.Write( targetType, SeverityType.Error,  "2001", 
                           "The {0} type does not have a nested type named 'Factory'.", 
                           targetType.Name ); 
        } 
        
        // ...
    } 
}

[BusinessRulePatternValidation]
public abstract class BusinessRule
{
  // ...
}
```


