---
uid: caching-dependencies
redirect_from:
  - caching/caching-dependencies
title: "Working with Cache Dependencies"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on working with cache dependencies in PostSharp, including adding string dependencies, adding object-oriented dependencies, and suspending the collection of cache dependencies. It includes code examples for each scenario."
---
# Working with Cache Dependencies

Cache dependencies have two major use cases. First, dependencies can act as a middle layer between the cached methods (typically the read methods) and the invalidating methods (typically the update methods) and therefore reduce the coupling between the read and update methods. Second, cache dependencies can be used to represent external dependencies, such as file system dependencies or SQL dependencies.

Compared to direct invalidation, using dependencies exhibits lower performance and higher resource consumption in the caching backend because of the need to store and synchronize the graph of dependencies. For details about direct invalidation, see <xref:caching-invalidation>. 


## Adding string dependencies

Eventually, all dependencies are represented as strings. Although we recommend using one of the strongly-typed approaches described below, it is good to understand how string dependencies work.


### Assigning a string dependency to a cached method and invalidating it

1. Add a call to the <xref:PostSharp.Patterns.Caching.ICachingContext.AddDependency(System.String)> method to the cached method. 


2. Add a call to the <xref:PostSharp.Patterns.Caching.CachingServices.Invalidation.Invalidate(System.String)> method to the invalidating method. 


> [!NOTE]
> Dependencies work correctly with recursive method calls. If a cached method `A` calls another cached method `B`, all dependencies of `B` are automatically dependencies of `A`, even if the return value of `B` was taken from the cache when `A` was being evaluated. 


### Example

In this example, the `GetValue` method assigns a string dependency to its cached return value. The `Update` method invalidates the dependency. This causes the related cached return value to be invalidated as well. 

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using PostSharp.Patterns.Caching;
using PostSharp.Patterns.Caching.Backends;

namespace PostSharp.Samples.Caching.StringDependencies
{
    class Database
    {
        private Dictionary<int, string> data = new Dictionary<int, string>();

        private static string GetValueDependencyString( int id ) => $"value:{id}";

        [Cache]
        public string GetValue( int id )
        {
            Console.WriteLine( $">> Retrieving {id} from the database..." );
            Thread.Sleep( 1000 );
            CachingServices.CurrentContext.AddDependency( GetValueDependencyString( id ) );
            return this.data[id];
        }

        public void Update( int id, string value )
        {
            this.data[id] = value;
            CachingServices.Invalidation.Invalidate( GetValueDependencyString( id ) );
        }
    }

    class Program
    {
        static void Main( string[] args )
        {
            CachingServices.DefaultBackend = new MemoryCachingBackend();

            Database db = new Database();

            db.Update( 1, "first" );

            Console.WriteLine( "Retrieving value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );

            Console.WriteLine( "Retrieving value of 1 for the 2nd time should NOT hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );

            db.Update( 1, "second" );

            Console.WriteLine( "Retrieving updated value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetValue( 1 ) );
        }
    }
}
```

The output of this sample is:

```text
Retrieving value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: first
Retrieving value of 1 for the 2nd time should NOT hit the database.
Retrieved: first
Retrieving updated value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: second
```


## Adding object-oriented dependencies through the ICacheDependency interface

Working with string dependencies can be error-prone because the code generating the string is duplicated in the invalidated and the invalidating method. A better approach is to encapsulate the cache key generation logic, that is, to represent the cache dependency as an object, and add some key-generation logic to this object.

If you own the source code of the class you want to use as a cache dependency, the easiest approach is to implement the <xref:PostSharp.Patterns.Caching.Dependencies.ICacheDependency> interface. 

> [!NOTE]
> This approach can be used to implement support for other kinds of dependencies, such as file system dependencies or SQL dependencies.


### Example

In the following example, the `Customer` class represents a business entity. Instances of this class are cached. At the same time, they serve as object dependencies, so the `Customer` class implements the <xref:PostSharp.Patterns.Caching.Dependencies.ICacheDependency> interface. The `GetValue` method assigns an object dependency of type `Customer` to its cached return value. The `Update` method invalidates the dependency. This causes the related cached return value to be invalidated as well. 

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using PostSharp.Patterns.Caching;
using PostSharp.Patterns.Caching.Backends;
using PostSharp.Patterns.Caching.Dependencies;

namespace PostSharp.Samples.Caching.ICacheDependencies
{
    class Customer : ICacheDependency
    {
        public int Id { get; set; }

        public string Name { get; set; }

        public bool Equals( ICacheDependency other )
        {
            Customer otherCustomer = other as Customer;
            return otherCustomer != null && this.Id == otherCustomer.Id;
        }

        public string GetCacheKey() => $"{nameof(Customer)}:{this.Id}";
    }

    class Database
    {
        private Dictionary<int, Customer> customers = new Dictionary<int, Customer>();

        [Cache]
        public Customer GetCustomer( int id )
        {
            Console.WriteLine( $">> Retrieving {id} from the database..." );
            Thread.Sleep( 1000 );
            Customer customer = this.customers[id];
            CachingServices.CurrentContext.AddDependency( customer );
            return customer;
        }

        public void Update( Customer customer )
        {
            this.customers[customer.Id] = customer;
            CachingServices.Invalidation.Invalidate( customer );
        }
    }

    class Program
    {
        static void Main( string[] args )
        {
            CachingServices.DefaultBackend = new MemoryCachingBackend();

            Database db = new Database();

            db.Update( new Customer() {Id = 1, Name = "Alice"} );

            Console.WriteLine( "Retrieving value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetCustomer( 1 ).Name );

            Console.WriteLine( "Retrieving value of 1 for the 2nd time should NOT hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetCustomer( 1 ).Name );

            db.Update( new Customer() {Id = 1, Name = "Bob"} );

            Console.WriteLine( "Retrieving updated value of 1 for the 1st time should hit the database." );
            Console.WriteLine( "Retrieved: " + db.GetCustomer( 1 ).Name );
        }
    }
}
```

The output of this sample is:

```text
Retrieving value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: Alice
Retrieving value of 1 for the 2nd time should NOT hit the database.
Retrieved: Alice
Retrieving updated value of 1 for the 1st time should hit the database.
>> Retrieving 1 from the database...
Retrieved: Bob
```


## Adding object-oriented dependencies through a formatter

The previous approach requires implementing an interface in the source code of the business entity. If you cannot modify the source code of a dependency class, the best approach is to implement a formatter for this class and to register it.

See <xref:caching-keys> for details. 


## Suspending the collection of cache dependencies

A new caching context, accessible through the <xref:PostSharp.Patterns.Caching.CachingServices.CurrentContext>, is created for each cached method. The caching context is propagated along all invoked methods. It is implemented using <xref:System.Threading.AsyncLocal`1> on platforms that support it. Otherwise, it is implemented using <xref:System.Runtime.Remoting.Messaging.CallContext.LogicalGetData(System.String)>. 

When a parent cached method calls a child cached method, the dependencies of the child methods are automatically added to the parent method, even if the child method was actually not executed because its result was found in cache. Therefore, invalidating a child method automatically invalidates the parent method, which is usually an intuitive and desirable behavior.

There are cases where propagating the caching context from the parent to the child methods (and therefore the collection of child dependencies into the parent context) is not desirable. For instance, if the parent method runs an asynchronous child task using `Task.Run` and does not wait for its completion, then it is likely that the dependencies of methods called in the child task should not be propagated to the parent (the child task could be considered a side effect of the parent method, and should not affect caching). Undesired dependencies would not make the program incorrect, but they would make it less efficient. 

To suspend the collection of dependencies in the current context and in all children contexts, you can use the <xref:PostSharp.Patterns.Caching.CachingServices.SuspendDependencyPropagation> method with a `using` construct. 


### Example

In the next example, the dependencies of `ChildMethod` (a side-effect method calling the cached method `ToString`) are not propagated to the parent `CachedMethod`. 

```csharp
[Cache]
int CachedMethod()
{
    using ( CachingServices.SuspendDependencyPropagation() )
    {
        Task.Run( ChildMethod );
    }

    return 0;
}

void ChildMethod()
{
    Console.WriteLine( "ChildMethod:" + this.ToString() );
}

[Cache]
public override string ToString()
{
    CachingServices.CurrentContext.AddDependency( "MyDependency" );
    return "{MyObject}";
}
```


